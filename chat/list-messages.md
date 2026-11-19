# List Messages (`GET /conversations/:id/messages`)

Retrieves messages from a conversation in reverse chronological order (newest first) from ScyllaDB (`messages` table), supporting TimeUUID cursor pagination.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/conversations/:id/messages`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 300 requests per minute per authenticated user
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
| `id` | `string` | Yes | UUID of the conversation |

### Query Parameters
| Parameter | Type | Required | Default | Constraints | Description |
|---|---|---|---|---|---|
| `limit` | `integer` | No | `30` | Min `1`, Max `100` | Maximum number of messages to return per page |
| `before` | `string` | No | `None` | Valid TimeUUID format | Message cursor ID; returns messages created before this TimeUUID |

---

## Behavior & Storage Operations

1. **UUID Validation**: Validates `id` and optional `before` as UUID formats.
2. **Membership Verification**: Verifies the conversation exists and that `req.user.userId` is present in `conversations.member_ids`.
3. **Clustered Read**: Queries ScyllaDB with clustering order `message_id DESC`:
   - Initial fetch: `WHERE conversation_id = ? LIMIT ?`
   - Paginated fetch: `WHERE conversation_id = ? AND message_id < ? LIMIT ?`
4. **Pagination State**: Sets `hasMore = (messages.length === limit)`.

---

## Response

### 200 OK
Messages retrieved successfully.

```json
{
  "data": [
    {
      "id": "be15a7f0-e35d-11f1-abbd-a1b88e74a509",
      "conversationId": "e2f7b8a0-1234-5678-9abc-def012345678",
      "senderId": "22222222-2222-4222-8222-222222222222",
      "body": "Hey, are you free to chat?",
      "createdAt": "2026-11-19T14:35:10.000Z",
      "editedAt": null,
      "deleted": false
    },
    {
      "id": "a9041230-e35d-11f1-abbd-a1b88e74a509",
      "conversationId": "e2f7b8a0-1234-5678-9abc-def012345678",
      "senderId": "11111111-1111-4111-8111-111111111111",
      "body": "Hi there! Welcome to Nebron.",
      "createdAt": "2026-11-19T14:30:25.000Z",
      "editedAt": null,
      "deleted": false
    }
  ],
  "hasMore": false
}
```

---

## Error Responses

### 400 Bad Request (`invalid_uuid`)
The conversation `id` or cursor `before` is not a valid UUID string.

```json
{
  "error": {
    "code": "invalid_uuid",
    "message": "Invalid conversation id"
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

### 403 Forbidden (`not_a_member`)
The authenticated user is not a participant in this conversation.

```json
{
  "error": {
    "code": "not_a_member",
    "message": "User is not a member of this conversation"
  }
}
```

### 404 Not Found (`conversation_not_found`)
Conversation does not exist in ScyllaDB.

```json
{
  "error": {
    "code": "conversation_not_found",
    "message": "Conversation not found"
  }
}
```

### 429 Too Many Requests (`rate_limit_exceeded`)
User exceeded the rate limit threshold of 300 requests per minute.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded, retry in 8 seconds"
  }
}
```

---

## Curl Example

```bash
curl -i "http://localhost:4003/conversations/e2f7b8a0-1234-5678-9abc-def012345678/messages?limit=20" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```
