# Internal Space Voice Channel Join (`POST /internal/spaces/:spaceId/channels/:chId/join`)

Internal endpoint called by `spaces-service` to verify voice channel permissions and issue a LiveKit access token for room `space_{spaceId}_channel_{chId}`.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/internal/spaces/:spaceId/channels/:chId/join`
- **Authentication**: `X-Internal-Token: <INTERNAL_TOKEN>` (or `Authorization: Bearer <INTERNAL_TOKEN>`)
- **Service**: `voice-service` (Port: `4004`)

---

## Request Body
| Field | Type | Required | Description |
|---|---|---|---|
| `userId` | `string` | Yes | Authenticated member ID joining the voice channel |

---

## Behavior & Response

1. Looks up channel permissions in Redis (`space:perm:{spaceId}:{chId}:{userId}`, 60s TTL) or fetches `GET /spaces/:spaceId/channels/:chId/permissions/:userId` from `spaces-service` using `INTERNAL_TOKEN`.
2. Requires `isMember === true`, `VIEW_CHANNEL`, and `CONNECT` (`403 missing_permission` otherwise). If `spaces-service` is unreachable, fails closed (`503 service_unavailable`).
3. Checks `SPEAK` bit to set LiveKit publish grant (`canPublish`).
4. Mints a LiveKit token scoped to `roomName = "space_{spaceId}_channel_{chId}"` and returns `200 OK`:
```json
{
  "token": "<livekit-jwt>",
  "roomName": "space_<spaceId>_channel_<chId>",
  "wsUrl": "<livekit-ws-url>",
  "canPublish": true
}
```
