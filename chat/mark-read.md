# Mark Messages as Read (`POST /conversations/:id/read`)

Records a read receipt up to a specific message TimeUUID and resets the caller's unread counter to 0.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/conversations/:id/read`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 120 requests per minute per authenticated user
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |
| `Content-Type` | `application/json` | Yes | Specifies request payload format |

### Path Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | `string` | Yes | UUID of the conversation |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `messageId` | `string` | Yes | Valid TimeUUID format | TimeUUID of the latest message read by the user |

### Example Request Body
```json
{
  "messageId": "be15a7f0-e35d-11f1-abbd-a1b88e74a509"
}
```

---

## Behavior & Storage Operations

1. **UUID Validation**: Validates `id` and `messageId` as valid UUIDs.
2. **Membership Verification**: Verifies `req.user.userId` belongs to `conversations.member_ids`.
3. **Message Existence**: Confirms that `messageId` belongs to `messages` for this conversation (`conversation_id = id AND message_id = messageId`).
4. **Receipt Upsert**: Checks existing `receipts` record for `(conversation_id, user_id)`. If absent or if `messageId > last_read_id`, updates `last_read_id` and sets `last_read_at = now()`.
5. **Unread Counter Reset**: Calls `clearUnread` to reset `unread_count = 0` in `user_conversations` for `(user_id, conversation_id)`.

---

## Response

### 204 No Content
Receipt recorded and unread counter reset. No response body is returned.

---

## Error Responses

### 400 Bad Request (`message_not_in_conversation`)
The provided `messageId` does not exist in the specified conversation.

```json
{
  "error": {
    "code": "message_not_in_conversation",
    "message": "Message not found in this conversation"
  }
}
```

### 400 Bad Request (`invalid_uuid`)
The conversation `id` or `messageId` is not a valid UUID string.

```json
{
  "error": {
    "code": "invalid_uuid",
    "message": "Invalid message id"
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
Authenticated user is not a participant in this conversation.

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
User exceeded the rate limit threshold of 120 requests per minute.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded, retry in 30 seconds"
  }
}
```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4003/conversations/e2f7b8a0-1234-5678-9abc-def012345678/read \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "messageId": "be15a7f0-e35d-11f1-abbd-a1b88e74a509"
  }'
```
