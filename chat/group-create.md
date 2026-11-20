# Create Group Conversation (`POST /conversations/group`)

Creates a new multi-user group conversation in ScyllaDB with the authenticated user designated as the group owner.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/conversations/group`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 5 requests per minute per authenticated user
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
| `name` | `string` | Yes | 2–100 characters, no HTML, control characters stripped | Group display name |
| `memberIds` | `string[]` | Yes | Array of 1–99 unique UUIDs, cannot contain caller | Initial member IDs to add to the group |

### Example Request Body
```json
{
  "name": "Backend Engineering",
  "memberIds": [
    "22222222-2222-4222-8222-222222222222",
    "33333333-3333-4333-8333-333333333333"
  ]
}
```

---

## Behavior & Storage Operations

1. **Input Sanitization & Validation**:
   - Strips control characters from `name` and trims whitespace.
   - Rejects names shorter than 2 or longer than 100 characters.
   - Rejects names containing HTML tags (`<...>` pattern).
   - Validates that `memberIds` is non-empty, contains valid UUIDs, has no duplicates, and does not include caller's `userId`.
   - Verifies total member count (caller + members) does not exceed 100.
2. **ScyllaDB Atomic & Denormalized Storage**:
   - Inserts row into `conversations` with `kind = 'group'`, `name`, `owner_id = callerId`, `member_count = 1 + memberIds.length`, and denormalized `member_ids` set.
   - Inserts atomic membership rows in `conversation_members`:
     - Caller as role `'owner'`.
     - Each added user as role `'member'`.
   - Seeds `user_conversations` for all members with `other_user_id = null` and `unread_count = 0`.
3. **Real-Time Notification & Event Bus**:
   - Emits `conversation.created` frame to each initial member's personal channel `ws:user:{memberId}` so their active sockets automatically subscribe to `ws:conv:{conversationId}`.
   - Publishes `chat.conversation.created` event to Redis Stream `nebron.events`.

---

## Response

### 201 Created
Group conversation created successfully.

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "kind": "group",
    "name": "Backend Engineering",
    "ownerId": "11111111-1111-4111-8111-111111111111",
    "memberIds": [
      "11111111-1111-4111-8111-111111111111",
      "22222222-2222-4222-8222-222222222222",
      "33333333-3333-4333-8333-333333333333"
    ],
    "memberCount": 3,
    "createdAt": "2026-11-20T19:00:00.000Z"
  }
}
```

---

## Error Responses

### 400 Bad Request (`invalid_payload`)
Group name too short/long, contains HTML, duplicate members provided, or caller included themselves in `memberIds`.

```json
{
  "error": {
    "code": "invalid_payload",
    "message": "Group name must be 2-100 characters"
  }
}
```

### 400 Bad Request (`group_too_large`)
Total members exceed the v1 cap of 100 members.

```json
{
  "error": {
    "code": "group_too_large",
    "message": "Group cannot exceed 100 members"
  }
}
```

### 401 Unauthorized (`invalid_token`)
Missing or invalid authentication token.

```json
{
  "error": {
    "code": "invalid_token",
    "message": "Authorization header is required"
  }
}
```

### 429 Too Many Requests (`rate_limit_exceeded`)
User exceeded the rate limit of 5 group creations per minute.
