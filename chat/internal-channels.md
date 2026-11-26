# Internal Provision Space Channel (`POST /internal/channels`)

Internal service-to-service endpoint called by `spaces-service` when creating a `text` channel inside a Space. Idempotently provisions a `kind = 'channel'` conversation in ScyllaDB with deterministic UUIDv5.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/internal/channels`
- **Authentication**: `X-Internal-Token: <INTERNAL_TOKEN>` (or `Authorization: Bearer <INTERNAL_TOKEN>`)
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Body Parameters
| Field | Type | Required | Constraints |
|---|---|---|---|
| `spaceId` | `string` | Yes | Space ObjectId |
| `channelId` | `string` | Yes | Channel ObjectId |
| `name` | `string` | Yes | 1–100 chars |

---

## Behavior & Response

1. Derives deterministic UUIDv5 from `"nebron:space-channel:{spaceId}:{channelId}"`.
2. Inserts or returns existing row in ScyllaDB `conversations` (`kind = 'channel'`, `space_id`, `channel_id`, `name`).
3. Returns `201 Created`:
```json
{
  "conversationId": "<uuid>"
}
```
4. Channel messages, reads, typing, reactions, and WebSocket `channel.subscribe` authorize access via cached `spaces-service` channel permissions (`space:perm:{spaceId}:{channelId}:{userId}`, 60s TTL, invalidated on `spaces.*` events).
