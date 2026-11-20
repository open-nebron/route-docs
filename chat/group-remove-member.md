# Remove Member or Leave Group (`DELETE /conversations/:id/members/:userId`)

Removes a member from a group conversation or allows an authenticated user to leave the group.

---

## Route Overview

- **Method**: `DELETE`
- **Path**: `/conversations/:id/members/:userId`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 20 requests per minute per authenticated user
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
| `userId` | `string` (UUID) | Yes | User UUID to remove from group |

---

## Behavior & Permissions

### Action Matrix
| Caller Role | Target `userId` | Permitted? | Result |
|---|---|---|---|
| Member | Self (`userId === caller`) | Yes | Member leaves the group |
| Owner | Self (`userId === caller`) | Yes | Owner leaves; ownership transfers to earliest joined member |
| Owner | Member (`userId !== caller`) | Yes | Owner removes/kicks member |
| Member | Other Member (`userId !== caller`) | **No (403)** | Forbidden (`not_owner`) |

### Owner Leaving Semantics
- If the owner leaves and other members remain, ownership transfers automatically to the member with the earliest `joined_at` timestamp in `conversation_members`.
- If the owner leaves and was the last member in the group, the conversation and its records are deleted entirely.

### Storage & Event Operations
1. Deletes row from `conversation_members`.
2. Deletes row from `user_conversations` for the leaving/removed user.
3. Removes user ID from `conversations.member_ids` set.
4. Decrements `conversations.member_count`.
5. Emits `conversation.member_removed` to `ws:conv:{id}` and `ws:user:{userId}` (causing the removed user's socket to unsubscribe from `ws:conv:{id}`).
6. Publishes `chat.conversation.member_removed` (or `chat.conversation.deleted`) to Redis Stream `nebron.events`.

---

## Response

### 204 No Content
Member removed or successfully left the group.

---

## Error Responses

### 400 Bad Request (`not_a_member_of_group`)
Target user is not a member of the group.

```json
{
  "error": {
    "code": "not_a_member_of_group",
    "message": "User is not a member of this group"
  }
}
```

### 403 Forbidden (`not_owner`)
Caller is not the group owner and attempted to remove another user.

```json
{
  "error": {
    "code": "not_owner",
    "message": "Only the group owner can remove members"
  }
}
```

### 404 Not Found (`conversation_not_found`)
Conversation does not exist.
