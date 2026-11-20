# Rename Group Conversation (`PATCH /conversations/:id`)

Updates the display name of a group conversation. Restricted to the group owner.

---

## Route Overview

- **Method**: `PATCH`
- **Path**: `/conversations/:id`
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

### Path Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | `string` (UUID) | Yes | Group conversation UUID |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `name` | `string` | Yes | 2–100 characters, no HTML | New display name for the group |

### Example Request Body
```json
{
  "name": "Core Platform Engineering"
}
```

---

## Behavior & Storage Operations

1. **Authorization**: Caller must be the group owner (`role === 'owner'`). Non-owners receive `403 not_owner`.
2. **Name Sanitization**: Control characters stripped, HTML tags rejected, whitespace trimmed, length checked (2–100 chars).
3. **ScyllaDB Update**: Updates `name` in `conversations` table.
4. **Real-Time Notification**:
   - Emits `conversation.renamed` to `ws:conv:{id}` with `newName` and `renamedBy`.
   - Publishes `chat.conversation.renamed` to Redis Stream `nebron.events`.

---

## Response

### 200 OK
Group successfully renamed.

```json
{
  "data": {
    "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "kind": "group",
    "name": "Core Platform Engineering",
    "ownerId": "11111111-1111-4111-8111-111111111111",
    "memberIds": [
      "11111111-1111-4111-8111-111111111111",
      "22222222-2222-4222-8222-222222222222"
    ],
    "memberCount": 2,
    "createdAt": "2026-11-20T19:00:00.000Z"
  }
}
```

---

## Error Responses

### 400 Bad Request (`invalid_payload`)
Name is shorter than 2 characters, longer than 100 characters, or contains HTML.

```json
{
  "error": {
    "code": "invalid_payload",
    "message": "Group name must be 2-100 characters"
  }
}
```

### 403 Forbidden (`not_owner`)
Caller is not the group owner.

```json
{
  "error": {
    "code": "not_owner",
    "message": "Only the group owner can rename the group"
  }
}
```

### 404 Not Found (`conversation_not_found`)
Conversation does not exist.
