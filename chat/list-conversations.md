# List Conversations (`GET /conversations`)

Retrieves the list of conversations for the authenticated user from ScyllaDB (`user_conversations`), ordered descending by `last_message_at`.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/conversations`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 300 requests per minute per authenticated user
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Query Parameters
| Parameter | Type | Required | Default | Constraints | Description |
|---|---|---|---|---|---|
| `limit` | `integer` | No | `30` | Min `1`, Max `100` | Maximum number of conversations to return |
| `before` | `string` | No | `None` | Valid ISO 8601 timestamp | Cursor timestamp; returns conversations prior to this date |

---

## Response

### 200 OK
Returns list of user conversations.

```json
{
  "data": [
    {
      "id": "e2f7b8a0-1234-5678-9abc-def012345678",
      "peerUserId": "22222222-2222-4222-8222-222222222222",
      "lastMessage": {
        "id": "be15a7f0-e35d-11f1-abbd-a1b88e74a509",
        "body": "Hey, are you free to chat?",
        "senderId": "22222222-2222-4222-8222-222222222222",
        "createdAt": "2026-11-19T14:35:10.000Z"
      },
      "unreadCount": 1,
      "lastMessageAt": "2026-11-19T14:35:10.000Z"
    },
    {
      "id": "c1a2b3c4-d5e6-7f8a-9b0c-1d2e3f4a5b6c",
      "peerUserId": "33333333-3333-4333-8333-333333333333",
      "lastMessage": null,
      "unreadCount": 0,
      "lastMessageAt": "2026-11-19T14:00:00.000Z"
    }
  ]
}
```

---

## Error Responses

### 400 Bad Request (`invalid_payload`)
Query parameters failed validation (e.g. invalid `limit` or malformed `before` ISO timestamp string).

```json
{
  "error": {
    "code": "invalid_payload",
    "message": "Invalid before timestamp"
  }
}
```

### 401 Unauthorized (`invalid_token`)
Missing, expired, untrusted signature, or invalid issuer/audience access token.

```json
{
  "error": {
    "code": "invalid_token",
    "message": "Authorization header is required"
  }
}
```

### 429 Too Many Requests (`rate_limit_exceeded`)
User exceeded the rate limit threshold of 300 requests per minute.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded, retry in 12 seconds"
  }
}
```

---

## Cursor Pagination Example

To retrieve the next page of conversations, pass the `lastMessageAt` of the last conversation item as the `before` query parameter:

```bash
curl -i "http://localhost:4003/conversations?limit=10&before=2026-11-19T14:00:00.000Z" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```
