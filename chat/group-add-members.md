# Add Members to Group (`POST /conversations/:id/members`)

Adds one or more new members to an existing group conversation. Restricted strictly to the group owner.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/conversations/:id/members`
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
| `userIds` | `string[]` | Yes | Array of 1–99 unique valid UUIDs | User UUIDs to add to the group |

### Example Request Body
```json
{
  "userIds": [
    "44444444-4444-4444-8444-444444444444"
  ]
}
```

---

## Behavior & Authorization

1. **Ownership Check**: Only the group owner (`role === 'owner'`) may add members. Non-owners receive `403 not_owner`.
2. **Cap Enforcement**: Ensures `currentMemberCount + userIds.length <= 100`. Exceeding returns `400 group_too_large`.
3. **Duplicate & Membership Check**: Validates that target users are not already members of the group. If any user is already a member, returns `400 already_member`.
4. **Storage Updates**:
   - Adds atomic rows in `conversation_members` with `role = 'member'`.
   - Appends user IDs to `conversations.member_ids` set.
   - Increments `conversations.member_count`.
   - Inserts `user_conversations` record for each new member.
5. **Real-Time Notification**:
   - Publishes `conversation.member_added` to `ws:user:{userId}` for each new member (triggering automatic WS subscription to `ws:conv:{conversationId}`).
   - Publishes `conversation.member_added` to `ws:conv:{id}` to notify all existing members.
   - Publishes `chat.conversation.member_added` to Redis Stream `nebron.events`.

---

## Response

### 200 OK
Members added successfully.

```json
{
  "data": {
    "conversationId": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "added": [
      "44444444-4444-4444-8444-444444444444"
    ],
    "memberCount": 4
  }
}
```

---

## Error Responses

### 400 Bad Request (`already_member`)
One or more users in `userIds` are already members of the group.

```json
{
  "error": {
    "code": "already_member",
    "message": "User 44444444-4444-4444-8444-444444444444 is already a member"
  }
}
```

### 400 Bad Request (`group_too_large`)
Adding the requested users would exceed the 100-member cap.

```json
{
  "error": {
    "code": "group_too_large",
    "message": "Group member cap of 100 exceeded"
  }
}
```

### 403 Forbidden (`not_owner`)
Caller is a member of the group but not the group owner.

```json
{
  "error": {
    "code": "not_owner",
    "message": "Only the group owner can add members"
  }
}
```

### 404 Not Found (`conversation_not_found`)
Conversation ID does not exist.
