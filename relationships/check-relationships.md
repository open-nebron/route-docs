# Batch Check Relationships (`GET /relationships/check`)

Internal microservice endpoint for batch querying relationship statuses. Used by `chat-service` and other internal services to silently drop messages from blocked senders or suppress typing/presence for muted users.

Protected by `X-Internal-Token` header. Responses include `Cache-Control: private, max-age=60`.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/relationships/check`
- **Authentication**: `X-Internal-Token` matching `INTERNAL_TOKEN` environment variable
- **Rate Limit**: 600 requests per minute per token
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `X-Internal-Token` | `<INTERNAL_TOKEN>` | Yes | Shared internal authentication token configured across microservices |

### Query Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `userIds` | string | Yes | Comma-separated list of target user IDs (e.g. `userIds=usr_1,usr_2,usr_3`) |

### Body Parameters
None.

---

## Response

### 200 OK
Batch lookup results mapped by user ID. Unknown user IDs return zero-state.

Headers:
`Cache-Control: private, max-age=60`

```json
{
  "data": {
    "usr_1": {
      "blocked": true,
      "muted": {
        "calls": false,
        "messages": false
      }
    },
    "usr_2": {
      "blocked": false,
      "muted": {
        "calls": true,
        "messages": true
      }
    },
    "usr_3": {
      "blocked": false,
      "muted": {
        "calls": false,
        "messages": false
      }
    }
  }
}
```

### 401 Unauthorized
Invalid or missing `X-Internal-Token` header.

```json
{
  "error": {
    "code": "invalid_internal_token",
    "message": "Invalid or missing internal token"
  }
}
```

### 503 Service Unavailable
`INTERNAL_TOKEN` is not configured on `core-api`.

```json
{
  "error": {
    "code": "internal_not_configured",
    "message": "Internal token verification not configured"
  }
}
```

### 429 Too Many Requests
Exceeded rate limit (600 requests/minute).
