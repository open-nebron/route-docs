# Unblock User (`DELETE /relationships/:userId/block`)

Unblocks a previously blocked user. Rejects self-actions, updates relationship state `{ blocked: false }`, and publishes the `core.relationship.unblocked` event to Redis Streams (`nebron.events`).

---

## Route Overview

- **Method**: `DELETE`
- **Path**: `/relationships/:userId/block`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"core"`)
- **Rate Limit**: 30 requests per minute per user
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Path Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `userId` | string | Yes | Target user ID (UUID, 24-character hex ObjectId, or `usr_` prefixed ID) |

### Body Parameters
None.

---

## Response

### 204 No Content
Relationship successfully set to unblocked. Event published to Redis Streams.

### 400 Bad Request
Target user ID is identical to caller's user ID, or format is invalid.

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

### 429 Too Many Requests
Exceeded rate limit (30 requests/minute).

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Too many requests. Please try again later."
  }
}
```

---

## Event Emitted
Redis stream `nebron.events`:
```json
{
  "type": "core.relationship.unblocked",
  "payload": {
    "fromUserId": "caller-user-id",
    "toUserId": "target-user-id",
    "at": "2026-11-19T20:00:00.000Z"
  }
}
```
