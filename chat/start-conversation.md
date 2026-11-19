# Start or Get Direct Conversation (`POST /conversations`)

Creates or retrieves a 1:1 direct messaging conversation between the authenticated caller and a peer user in ScyllaDB.

> [!NOTE]
> 1:1 conversation IDs are deterministically derived using UUIDv5: `uuidv5("nebron:conv:" + sorted(userA, userB), NAMESPACE)`. Calling this endpoint repeatedly with the same peer returns the existing conversation without duplicate records.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/conversations`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 10 requests per minute per authenticated user
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |
| `Content-Type` | `application/json` | Yes | Specifies request payload format |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `peerUserId` | `string` | Yes | Valid UUIDv4 format | UUID of the user to start a conversation with |

### Example Request Body
```json
{
  "peerUserId": "22222222-2222-4222-8222-222222222222"
}
```

---

## Behavior & Storage Operations

1. **Self-Chat Prevention**: Verifies `peerUserId !== req.user.userId`.
2. **Deterministic ID Generation**: Computes UUIDv5 over lexicographically sorted participant IDs.
3. **Conversations Upsert**: Inserts row into ScyllaDB `conversations` with `kind = 'direct'`, `member_ids = {currentUserId, peerUserId}`.
4. **User Conversations Seeding**: Idempotently seeds `user_conversations` rows for both participants with `unread_count = 0` and `other_user_id` populated.

---

## Response

### 200 OK
Conversation created or retrieved successfully.

```json
{
  "data": {
    "id": "e2f7b8a0-1234-5678-9abc-def012345678",
    "kind": "direct",
    "createdAt": "2026-11-19T14:30:00.000Z",
    "peerUserId": "22222222-2222-4222-8222-222222222222"
  }
}
```

---

## Error Responses

### 400 Bad Request (`cannot_start_conversation_with_self`)
Caller attempted to initiate a conversation with their own `userId`.

```json
{
  "error": {
    "code": "cannot_start_conversation_with_self",
    "message": "Cannot start conversation with yourself"
  }
}
```

### 400 Bad Request (`invalid_uuid`)
Provided `peerUserId` is not a valid UUID string.

```json
{
  "error": {
    "code": "invalid_uuid",
    "message": "peerUserId must be a valid UUID"
  }
}
```

### 400 Bad Request (`invalid_payload`)
Payload failed Zod validation (missing required fields or extraneous properties).

```json
{
  "error": {
    "code": "invalid_payload",
    "message": "Required"
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
User has exceeded the rate limit threshold of 10 requests per minute.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded, retry in 54 seconds"
  }
}
```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4003/conversations \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "peerUserId": "22222222-2222-4222-8222-222222222222"
  }'
```
