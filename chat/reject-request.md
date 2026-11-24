# Reject Message Request (`POST /conversations/:id/reject-request`)

Rejects a pending direct message request. The conversation is marked as `rejected`, audit events are dispatched, and future messages from the sender are silently dropped without alerting the sender.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/conversations/:id/reject-request`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 30 requests per minute per authenticated user
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
| `id` | `string` | Yes | UUID of the pending direct conversation |

### Body Parameters
None.

---

## Behavior & Semantics

1. **Conversation Lookup**: Loads conversation from ScyllaDB `conversations` table.
2. **State Verification**: Verifies `request_state === 'pending'`. Returns `400` if not pending.
3. **Recipient Authorization**: Verifies `req.user.userId` is the recipient. Returns `403` if called by sender or non-member.
4. **State Transition**: Updates `conversations.request_state = 'rejected'`.
5. **No Recipient Inbox Row**: Recipient never receives a `user_conversations` row, so the conversation never appears in their inbox or requests list.
6. **Event Publication**: Publishes `chat.conversation.request_rejected` event to RabbitMQ with payload `{ conversationId, rejectedBy, at }`.
7. **Sender Experience**: The sender's existing view remains untouched ("Awaiting response" badge), and future messages from the sender to this conversation receive silent `204 No Content` and are never stored or delivered.
8. **Response**: Returns `204 No Content`.

---

## Response

### 204 No Content
Request rejected successfully. No response body.

---

## Error Responses

### 400 Bad Request (`request_not_pending`)
Conversation is not in a pending request state.

```json
{
  "error": {
    "code": "request_not_pending",
    "message": "Conversation is not in a pending request state"
  }
}
```

### 403 Forbidden (`not_request_recipient`)
Caller is not the recipient of this message request.

```json
{
  "error": {
    "code": "not_request_recipient",
    "message": "Only the request recipient can reject this message request"
  }
}
```

### 404 Not Found (`conversation_not_found`)
Conversation ID does not exist.

```json
{
  "error": {
    "code": "conversation_not_found",
    "message": "Conversation not found"
  }
}
```
