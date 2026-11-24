# Accept Message Request (`POST /conversations/:id/accept-request`)

Accepts a pending direct message request, moving the conversation into the recipient's main inbox, backfilling user conversation state, fanning out all pending messages via WebSocket, and publishing an audit event.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/conversations/:id/accept-request`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 30 requests per minute per authenticated user
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Path Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | `string` | Yes | UUID of the pending direct conversation |

### Body Parameters
None.

---

## Behavior & Semantics

1. **Conversation Lookup**: Loads conversation from ScyllaDB `conversations` table.
2. **State Verification**: Verifies `request_state === 'pending'`. Returns `400` if not pending.
3. **Recipient Authorization**: Verifies `req.user.userId` is the recipient (cannot be the sender who initiated the request). Returns `403` if called by the sender or a non-participant.
4. **State Transition**: Updates `conversations.request_state = 'active'`.
5. **Inbox Row Creation**: Creates `user_conversations` row for recipient with `is_pending = false`, setting `last_message` and unread count from existing pending messages.
6. **WebSocket Fanout**: Delivers all pending messages to the recipient's connected WebSocket sessions as `message.new` frames.
7. **Event Publication**: Publishes `chat.conversation.request_accepted` event to RabbitMQ with payload `{ conversationId, acceptedBy, at }`.
8. **Response**: Returns `200 OK` with the active conversation object.

---

## Response

### 200 OK
```json
{
  "data": {
    "id": "e2f7b8a0-1234-5678-9abc-def012345678",
    "kind": "direct",
    "peerUserId": "11111111-1111-4111-8111-111111111111",
    "requestState": "active",
    "request_state": "active",
    "unreadCount": 1,
    "lastMessage": {
      "id": "msg-001",
      "body": "Hi there!",
      "senderId": "11111111-1111-4111-8111-111111111111",
      "createdAt": "2026-11-24T10:00:00.000Z"
    },
    "lastMessageAt": "2026-11-24T10:00:00.000Z"
  }
}
```

---

## Error Responses

### 400 Bad Request (`request_not_pending`)
Conversation is not in a pending request state (already active or rejected).

```json
{
  "error": {
    "code": "request_not_pending",
    "message": "Conversation is not in a pending request state"
  }
}
```

### 403 Forbidden (`not_request_recipient`)
Caller is not the recipient of this message request (sender cannot accept their own request).

```json
{
  "error": {
    "code": "not_request_recipient",
    "message": "Only the request recipient can accept this message request"
  }
}
```

### 404 Not Found (`conversation_not_found`)
Conversation ID does not exist.

```json
{
  "error": {
    "code": "conversation_not_found",
    "message": "Conversation not found"
  }
}
```
