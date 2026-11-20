# List Group Members (`GET /conversations/:id/members`)

Retrieves the list of members in a group conversation ordered ascending by join timestamp (`joined_at ASC`).

---

## Route Overview

- **Method**: `GET`
- **Path**: `/conversations/:id/members`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 60 requests per minute per authenticated user
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
| `id` | `string` (UUID) | Yes | Group conversation UUID |

### Query Parameters
| Parameter | Type | Required | Default | Constraints | Description |
|---|---|---|---|---|---|
| `limit` | `integer` | No | `50` | Min `1`, Max `100` | Maximum number of members to return |
| `before` | `string` | No | `None` | Valid ISO 8601 timestamp | Cursor timestamp |

---

## Behavior & Authorization

1. **Membership Verification**: Caller must be an active member of the conversation. Non-members receive `403 not_a_member`.
2. **ScyllaDB Query**: Fetches rows from `conversation_members` where `conversation_id = :id` ordered by clustering key `(joined_at ASC, user_id ASC)`.

---

## Response

### 200 OK
Returns list of members with roles and join timestamps.

```json
{
  "data": [
    {
      "userId": "11111111-1111-4111-8111-111111111111",
      "role": "owner",
      "joinedAt": "2026-11-20T19:00:00.000Z"
    },
    {
      "userId": "22222222-2222-4222-8222-222222222222",
      "role": "member",
      "joinedAt": "2026-11-20T19:00:00.000Z"
    },
    {
      "userId": "44444444-4444-4444-8444-444444444444",
      "role": "member",
      "joinedAt": "2026-11-20T19:05:00.000Z"
    }
  ],
  "hasMore": false
}
```

---

## Error Responses

### 403 Forbidden (`not_a_member`)
Caller is not a member of the conversation.

```json
{
  "error": {
    "code": "not_a_member",
    "message": "User is not a member of this conversation"
  }
}
```

### 404 Not Found (`conversation_not_found`)
Conversation does not exist.
