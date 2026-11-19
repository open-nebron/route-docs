# Send Message (`POST /conversations/:id/messages`)

Sends a direct message within an existing conversation, persists it to ScyllaDB, and atomically updates the unread counts and conversation summaries for both participants.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/conversations/:id/messages`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 30 requests per minute per authenticated user
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
| `id` | `string` | Yes | UUID of the target conversation |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `body` | `string` | Yes | 1–4000 characters; no `\u0000` | Text body of the message |

### Example Request Body
```json
{
  "body": "Hey Alex! Working on the new ScyllaDB schema today."
}
```

---

## Sanitization & Storage Operations

1. **Null Byte Check**: Rejects any string containing null bytes (`\u0000`).
2. **Control Character Stripping**: Strips ASCII control characters `[\u0001-\u0008\u000B\u000C\u000E-\u001F\u007F]`, while preserving whitespace, newlines (`\n`), and tabs (`\t`).
3. **Trimming & Length Verification**: Trims leading/trailing whitespace. Rejects empty messages (length 0) and strings exceeding 4000 characters.
4. **Membership Verification**: Verifies `req.user.userId` is a member of `conversations.member_ids`.
5. **Message Insertion**: Generates a new `TimeUuid` for `message_id` and saves to `messages` with `deleted = false`.
6. **User Conversation Index Updates**:
   - **Sender**: Updates `user_conversations` (`last_message_id`, `last_message_at`, `last_message_body`), unread count remains unchanged (`unreadDelta = 0`).
   - **Recipient**: Updates `user_conversations` (`last_message_id`, `last_message_at`, `last_message_body`), increments unread count by 1 (`unreadDelta = 1`).

---

## Response

### 201 Created
Message persisted and returned successfully.

```json
{
  "data": {
    "id": "be15a7f0-e35d-11f1-abbd-a1b88e74a509",
    "conversationId": "e2f7b8a0-1234-5678-9abc-def012345678",
    "senderId": "11111111-1111-4111-8111-111111111111",
    "body": "Hey Alex! Working on the new ScyllaDB schema today.",
    "createdAt": "2026-11-19T14:35:10.000Z",
    "editedAt": null,
    "deleted": false
  }
}
```

---

## Error Responses

### 400 Bad Request (`empty_message`)
Message body is empty or consists entirely of whitespace or control characters.

```json
{
  "error": {
    "code": "empty_message",
    "message": "Message body cannot be empty"
  }
}
```

### 400 Bad Request (`message_too_long`)
Message body exceeds the maximum length of 4000 characters.

```json
{
  "error": {
    "code": "message_too_long",
    "message": "Message cannot exceed 4000 characters"
  }
}
```

### 400 Bad Request (`invalid_payload`)
Payload failed Zod validation or contains null bytes (`\u0000`).

```json
{
  "error": {
    "code": "invalid_payload",
    "message": "Message cannot contain null bytes"
  }
}
```

### 400 Bad Request (`invalid_uuid`)
The conversation `id` in the URL path is not a valid UUID string.

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
User exceeded the rate limit threshold of 30 requests per minute.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded, retry in 22 seconds"
  }
}
```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4003/conversations/e2f7b8a0-1234-5678-9abc-def012345678/messages \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "Hey Alex! Working on the new ScyllaDB schema today."
  }'
```
