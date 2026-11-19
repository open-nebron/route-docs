# Get Relationship (`GET /relationships/:userId`)

Retrieves the caller's relationship status with a target user. If no relationship record exists in the database, returns a zero-state HTTP 200 rather than a 404.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/relationships/:userId`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"core"`)
- **Rate Limit**: 300 requests per minute per user
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

### 200 OK
Relationship state returned.

```json
{
  "blocked": false,
  "mute": {
    "calls": false,
    "messages": false,
    "until": null
  }
}
```

If blocked and muted:
```json
{
  "blocked": true,
  "mute": {
    "calls": true,
    "messages": true,
    "until": "2026-11-20T20:00:00.000Z"
  }
}
```

Note: If an active `muteUntil` has expired based on current server time, `calls` and `messages` are dynamically resolved to `false`.

### 400 Bad Request
Invalid user ID format.

```json
{
  "error": {
    "code": "invalid_uuid",
    "message": "Invalid user ID"
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
Exceeded rate limit (300 requests/minute).
