# Mute User (`POST /relationships/:userId/mute`)

Mutes calls and/or messages from another user, optionally until a specified future expiration timestamp. Merges into existing mute state, and publishes the `core.relationship.muted` event to Redis Streams (`nebron.events`).

---

## Route Overview

- **Method**: `POST`
- **Path**: `/relationships/:userId/mute`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"core"`)
- **Rate Limit**: 30 requests per minute per user
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |
| `Content-Type` | `application/json` | Yes | Application JSON payload |

### Path Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `userId` | string | Yes | Target user ID (UUID, 24-character hex ObjectId, or `usr_` prefixed ID) |

### Body Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `calls` | boolean | Optional | Whether to mute calls from this user |
| `messages` | boolean | Optional | Whether to mute messages from this user |
| `until` | string (ISO 8601) \| null | Optional | Expiration timestamp in UTC ISO format (must be in the future, or null for indefinite) |

At least one parameter (`calls: true`, `messages: true`, or `until: non-null`) must be provided.

```json
{
  "calls": true,
  "messages": true,
  "until": "2026-11-20T20:00:00.000Z"
}
```

---

## Response

### 200 OK
Mute state successfully updated.

```json
{
  "calls": true,
  "messages": true,
  "until": "2026-11-20T20:00:00.000Z"
}
```

### 400 Bad Request
Empty mute payload, past expiration timestamp, self-mute action, or invalid user ID.

Empty mute:
```json
{
  "error": {
    "code": "empty_mute",
    "message": "At least one mute field must be set"
  }
}
```

Past timestamp:
```json
{
  "error": {
    "code": "invalid_until",
    "message": "until must be a future timestamp"
  }
}
```

Self mute:
```json
{
  "error": {
    "code": "self_action_not_allowed",
    "message": "Self action is not allowed"
  }
}
```

### 401 Unauthorized
Missing, expired, or invalid JWT token.

```json
{
  "error": {
    "code": "invalid_token",
    "message": "Invalid Compact JWS"
  }
}
```

---

## Event Emitted
Redis stream `nebron.events`:
```json
{
  "type": "core.relationship.muted",
  "payload": {
    "fromUserId": "caller-user-id",
    "toUserId": "target-user-id",
    "mute": {
      "calls": true,
      "messages": true,
      "until": "2026-11-20T20:00:00.000Z"
    },
    "at": "2026-11-19T20:00:00.000Z"
  }
}
```
