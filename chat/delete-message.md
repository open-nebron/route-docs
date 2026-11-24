# Soft-Delete Message (`DELETE /conversations/:id/messages/:messageId`)

Soft-deletes a message by setting `deleted = true` in ScyllaDB. Only the sender of the message is permitted to delete it.

---

## Route Overview

- **Method**: `DELETE`
- **Path**: `/conversations/:id/messages/:messageId`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 10 requests per minute per authenticated user
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
| `messageId` | `string` | Yes | TimeUUID of the message to soft-delete |

### Body Parameters
None.

---

## Behavior & Storage Operations

1. **UUID Validation**: Validates `id` and `messageId` as valid UUID formats.
2. **Conversation & Membership Check**: Verifies that the conversation exists and that `req.user.userId` is a member of `conversations.member_ids`.
3. **Message Existence**: Confirms message exists in ScyllaDB `messages` table for `(conversation_id = id AND message_id = messageId)`.
4. **Ownership Verification**: Enforces that `message.senderId === req.user.userId`. If a user attempts to delete another user's message, rejects with HTTP 403 `not_message_owner`.
5. **Soft-Delete Flag**: Updates `deleted = true` in `messages`. Message content and audit metadata remain persisted in ScyllaDB for compliance and tombstones.
6. **Real-time Event Broadcast**: Publishes `message.deleted` to Redis channel `ws:conv:{id}` and `ws:user:{memberId}` so all connected clients instantly update their message lists and conversation previews without refreshing.

---

## Response

### 204 No Content
Message successfully soft-deleted. No response body is returned.

---

## Error Responses

### 400 Bad Request (`invalid_uuid`)
The conversation `id` or `messageId` is not a valid UUID string.

```json
{
  "error": {
    "code": "invalid_uuid",
    "message": "Invalid conversation id or message id"
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

### 403 Forbidden (`not_message_owner`)
Caller is not the author of this message.

```json
{
  "error": {
    "code": "not_message_owner",
    "message": "You can only delete your own messages"
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

### 404 Not Found (`message_not_found`)
Message does not exist in this conversation.

```json
{
  "error": {
    "code": "message_not_found",
    "message": "Message not found"
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
User exceeded the rate limit threshold of 10 requests per minute.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded, retry in 45 seconds"
  }
}
```

---

## Curl Example

```bash
curl -i -X DELETE http://localhost:4003/conversations/e2f7b8a0-1234-5678-9abc-def012345678/messages/be15a7f0-e35d-11f1-abbd-a1b88e74a509 \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```
