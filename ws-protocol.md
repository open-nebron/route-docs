# WebSocket Real-Time Messaging Protocol

## Overview

The Nebron Chat Service exposes a high-throughput, low-latency WebSocket endpoint for real-time 1:1 messaging, presence updates, and live typing indicators.

- **Route**: `GET /ws?token=<access_token>`
- **Protocol**: WebSocket (RFC 6455)
- **Base URL**: `ws://localhost:4003/ws` (TLS: `wss://<host>/ws`)
- **Transport Subscriptions**: Backed by Redis Pub/Sub channels (`ws:conv:{id}`, `ws:user:{id}`, `ws:signal:{id}`).

---

## 1. Connection & Authentication Handshake

### 1.1 Query Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `token` | string | **Yes** | RS256 Bearer JWT issued by `auth-service` |

### 1.2 Verification Pipeline

1. **JWT Verification**:
   - Signature algorithm: `RS256`
   - Issuer (`iss`): `nebron-auth`
   - Audience (`aud`): Must include `chat`
   - Subject (`sub`): Authenticated user UUID
   - Session ID (`sessionId` or `sid`): Session identifier
   - Failure: Socket closed immediately with code **`4401`** (`invalid_token`).
2. **Session Revocation Check**:
   - Checks Redis key `revoked_sessions:{sessionId}`.
   - If present: Socket closed immediately with code **`4403`** (`session_revoked`).
3. **Connection Concurrency Limit**:
   - Maximum **5** concurrent sockets per user.
   - On the 6th connection, the oldest active connection for that user is terminated with code **`4408`** (`too_many_connections`).
4. **Frame Size Limit**:
   - Maximum frame size is **32 KB** (32,768 bytes).
   - Any frame exceeding this size results in socket termination with code **`4409`** (`frame_too_large`).

### 1.3 Ready Event

Upon successful connection, the server registers subscriptions, queries initial relationship block states, updates presence, and sends the `ready` event:

```json
{
  "type": "ready",
  "payload": {
    "sessionId": "sess_8f2b3e4a",
    "userId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "limits": {
      "messagesPerSec": 10,
      "typingPerSec": 10,
      "readPerSec": 20,
      "maxFrameBytes": 32768
    }
  }
}
```

---

## 2. Message Format & Ack ID Semantics

All frames are JSON objects. Client-originated frames can optionally provide an `id` field (string), which the server echoes as `ackId` in its direct response/ack.

### Frame Envelope:
```json
{
  "type": "<message_type>",
  "payload": { ... },
  "id": "client-correlation-id"
}
```

---

## 3. Client-to-Server Messages

### 3.1 `ping`
Heartbeat check. Responds with `pong`.

**Request**:
```json
{
  "type": "ping",
  "payload": {
    "data": "1732050000000"
  },
  "id": "ping-1"
}
```

**Response (`pong`)**:
```json
{
  "type": "pong",
  "payload": {
    "data": "1732050000000"
  },
  "ackId": "ping-1"
}
```

---

### 3.2 `message.send`
Send a new direct message.

**Payload Schema**:
| Field | Type | Required | Description |
|---|---|---|---|
| `conversationId` | string (UUID) | **Yes** | 1:1 conversation UUID |
| `body` | string | **Yes** | 1–4000 characters. Null bytes forbidden. Strips non-tab/newline controls. |
| `clientId` | string | *Optional* | Client-generated UUID for 5-minute idempotency deduplication |

**Request**:
```json
{
  "type": "message.send",
  "payload": {
    "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
    "body": "Hey, are you free for a call?",
    "clientId": "f9b8c2d1-e35d-11f1-abbd-a1b88e74a509"
  },
  "id": "msg-send-01"
}
```

**Response (`message.sent` Ack)**:
```json
{
  "type": "message.sent",
  "payload": {
    "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
    "clientId": "f9b8c2d1-e35d-11f1-abbd-a1b88e74a509",
    "message": {
      "id": "be15a7f0-e35d-11f1-abbd-a1b88e74a509",
      "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
      "senderId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
      "body": "Hey, are you free for a call?",
      "createdAt": "2026-11-19T22:00:00.000Z",
      "editedAt": null,
      "deleted": false
    },
    "delivered": true
  },
  "ackId": "msg-send-01"
}
```

*Note on Block Enforcement*:
If the recipient has blocked the sender, the message is dropped silently. The ack returns `"delivered": false` without `message`, and no data is written to ScyllaDB or delivered to the peer:
```json
{
  "type": "message.sent",
  "payload": {
    "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
    "clientId": "f9b8c2d1-e35d-11f1-abbd-a1b88e74a509",
    "delivered": false
  },
  "ackId": "msg-send-01"
}
```

---

### 3.3 `message.read`
Mark conversation as read up to a specified message.

**Payload Schema**:
| Field | Type | Required | Description |
|---|---|---|---|
| `conversationId` | string (UUID) | **Yes** | Conversation identifier |
| `messageId` | string (TimeUUID) | **Yes** | TimeUUID of the last read message |

**Request**:
```json
{
  "type": "message.read",
  "payload": {
    "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
    "messageId": "be15a7f0-e35d-11f1-abbd-a1b88e74a509"
  },
  "id": "read-ack-01"
}
```

**Response (`message.read.ack`)**:
```json
{
  "type": "message.read.ack",
  "payload": {
    "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
    "messageId": "be15a7f0-e35d-11f1-abbd-a1b88e74a509"
  },
  "ackId": "read-ack-01"
}
```

---

### 3.4 `typing.start` / `typing.stop`
Emit typing state.

**Request**:
```json
{
  "type": "typing.start",
  "payload": {
    "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a"
  }
}
```

Typing automatically times out after **5 seconds** of inactivity, automatically emitting `typing: false` to the recipient.

---

## 4. Server-to-Client Push Events

### 4.1 `message.new`
Pushed to recipient sockets when a message arrives.

```json
{
  "type": "message.new",
  "payload": {
    "senderId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "message": {
      "id": "be15a7f0-e35d-11f1-abbd-a1b88e74a509",
      "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
      "senderId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
      "body": "Hey, are you free for a call?",
      "createdAt": "2026-11-19T22:00:00.000Z",
      "editedAt": null,
      "deleted": false
    }
  }
}
```

### 4.2 `typing.update`
Pushed to recipient sockets when a peer starts or stops typing.

```json
{
  "type": "typing.update",
  "payload": {
    "conversationId": "e4f8a61b-90a3-5c24-9b2f-7848df74f07a",
    "userId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "typing": true
  }
}
```

### 4.3 `presence.update`
Pushed across mutual conversation channels when a peer's aggregate socket count transitions between 0 and 1.

```json
{
  "type": "presence.update",
  "userId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "online": true
}
```
Or when going offline:
```json
{
  "type": "presence.update",
  "userId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "online": false,
  "lastSeenAt": "2026-11-19T22:04:12.890Z"
}
```

---

## 5. Error Frames

When a message frame fails validation, is rate limited, or causes an error without terminating the socket connection:

```json
{
  "type": "error",
  "payload": {
    "code": "rate_limited",
    "message": "Rate limit exceeded"
  },
  "ackId": "req-2"
}
```

### Common Error Codes:
- `rate_limited`: Rate limit exceeded (send: 10/s, typing: 10/s, read: 20/s)
- `invalid_payload`: Payload missing required fields, malformed JSON, or contains null bytes
- `not_a_member`: User is not a participant in the specified conversation
- `empty_message`: Message body is empty or whitespace only
- `message_too_long`: Message body exceeds 4000 characters
- `message_not_in_conversation`: Referenced messageId does not exist in the conversation
