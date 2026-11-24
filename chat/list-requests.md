# List Pending Message Requests (`GET /conversations/requests`)

Lists all pending direct message requests where the authenticated user is the recipient.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/conversations/requests`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"chat"`)
- **Rate Limit**: 60 requests per minute per authenticated user
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Query Parameters
None.

### Body Parameters
None.

---

## Behavior & Semantics

1. **Lookup**: Scans or queries ScyllaDB `conversations` where `request_state = 'pending'` and `recipient = req.user.userId`.
2. **Payload Enrichment**: Returns each conversation with:
   - `id`: Conversation UUID
   - `kind`: `'direct'`
   - `senderId`: UUID of the sender who initiated the request
   - `peerUserId`: Same as `senderId` for UI convenience
   - `requestState`: `'pending'`
   - `lastMessage`: Preview of the first/latest message in the request
   - `lastMessageAt`: ISO timestamp of latest message
   - `createdAt`: ISO timestamp when request conversation was created
3. **Response**: Returns `200 OK` with `{ "data": Conversation[] }`.

---

## Response

### 200 OK
```json
{
  "data": [
    {
      "id": "e2f7b8a0-1234-5678-9abc-def012345678",
      "kind": "direct",
      "senderId": "11111111-1111-4111-8111-111111111111",
      "peerUserId": "11111111-1111-4111-8111-111111111111",
      "requestState": "pending",
      "request_state": "pending",
      "unreadCount": 0,
      "lastMessage": {
        "id": "msg-001",
        "body": "Hey! Would love to connect regarding your project.",
        "senderId": "11111111-1111-4111-8111-111111111111",
        "createdAt": "2026-11-24T10:00:00.000Z"
      },
      "lastMessageAt": "2026-11-24T10:00:00.000Z",
      "createdAt": "2026-11-24T10:00:00.000Z"
    }
  ]
}
```

---

## Error Responses

### 401 Unauthorized (`invalid_token`)
Missing, expired, or invalid JWT access token.
