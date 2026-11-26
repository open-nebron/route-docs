# Space Channels, Categories & Voice Join

Manage channel categories, text/voice channels, channel-level permission overrides, and voice channel token minting.

---

## Category Endpoints (`MANAGE_CHANNELS`)

- **`POST /spaces/:id/categories`**: Body `{ "name": string (1–100), "position"?: number (0–1000) }` -> `201 { "category": { ... } }`
- **`PATCH /spaces/:id/categories/:catId`**: Body `{ "name"?, "position"? }` -> `200 { "category": { ... } }`
- **`DELETE /spaces/:id/categories/:catId`**: Deletes category and resets `categoryId: null` on child channels -> `204 No Content`

---

## Channel Endpoints

### `GET /spaces/:id/channels`
Lists channels in the space ordered by `position` ASC, filtered to channels where the caller has `VIEW_CHANNEL`.
- **Response (`200 OK`)**: `{ "channels": [...] }`

### `POST /spaces/:id/channels`
Creates a text or voice channel. Requires `MANAGE_CHANNELS` (and validates `categoryId` belongs to `:id` -> `400 invalid_category`).
For `kind: "text"`, calls `chat-service` `POST /internal/channels` to provision a deterministic `conversationId`. Emits `spaces.channel.created`.

**Request Body**
| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | Yes | 1–100 chars |
| `kind` | `'text' \| 'voice'` | Yes | Channel type |
| `categoryId` | `string \| null` | No | Optional category ObjectId within space |
| `topic` | `string \| null` | No | Max 500 chars |
| `position` | `number` | No | Sort order `0–1000` |
| `permissionOverrides` | `Array<{ targetType, targetId, allow, deny }>` | No | Max 100 overrides |

- **Response (`201 Created`)**: `{ "channel": { ... } }`

### `PATCH /spaces/:id/channels/:chId`
Updates channel metadata or `permissionOverrides`. Editing metadata requires `MANAGE_CHANNELS`; editing `permissionOverrides` allows `MANAGE_CHANNELS` or `MANAGE_ROLES`. Emits `spaces.channel.updated`.
- **Response (`200 OK`)**: `{ "channel": { ... } }`

### `DELETE /spaces/:id/channels/:chId`
Deletes channel, emits `spaces.channel.deleted`, and returns `204 No Content`.

---

## Voice Channel Join

### `POST /spaces/:id/channels/:chId/voice/join`
Joins a space voice channel:
1. Verifies membership, channel existence, and `channel.kind === 'voice'` (`400 invalid_channel_kind` for text channels).
2. Verifies caller has both `VIEW_CHANNEL` and `CONNECT` permissions on `:chId`.
3. Delegates to `voice-service` `POST /internal/spaces/:spaceId/channels/:chId/join` with `X-Internal-Token` and `{ "userId": callerId }`.
4. Returns LiveKit connection payload (`200 OK`): `{ "token", "roomName", "wsUrl", "canPublish" }`.
