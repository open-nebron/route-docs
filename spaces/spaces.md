# Spaces Management (`POST/GET /spaces`, `GET/PATCH/DELETE /spaces/:id`)

Manage Discord-style community spaces, including default role/channel provisioning, membership queries, settings updates, and cascading deletion.

---

## Service Overview

- **Service**: `spaces-service` (Port: `4005`)
- **Authentication**: `Bearer <accessToken>` (RS256 JWT; audience includes `"spaces"` or `"core"`)
- **Platform Ban Check**: Authenticated requests check Redis key `user:banned:{userId}`; banned users receive `403 Forbidden` (`account_banned`).

---

## Endpoints

### 1. `POST /spaces`
Creates a new space, seeds the `@everyone` default role (`DEFAULT_EVERYONE_PERMISSIONS`), creates `Text Channels` and `Voice Channels` categories, creates `#general` (text) and `General` (voice), provisions a ScyllaDB conversation in `chat-service` via `POST /internal/channels`, and emits `spaces.space.created`.

**Request Body**
| Field | Type | Required | Constraints |
|---|---|---|---|
| `name` | `string` | Yes | 2–100 chars after sanitization |
| `description` | `string \| null` | No | Max 500 chars |

**Response (`201 Created`)**
```json
{
  "space": {
    "id": "6745...",
    "name": "Nebron Builders",
    "iconKey": null,
    "bannerKey": null,
    "description": "Engineering space",
    "ownerId": "...",
    "memberCount": 1,
    "defaultRoleId": "6745...",
    "createdAt": "2026-11-26T08:00:00.000Z",
    "updatedAt": "2026-11-26T08:00:00.000Z"
  },
  "categories": [...],
  "channels": [...],
  "roles": [...],
  "myPermissions": "..."
}
```
If `chat-service` fails to provision `#general`, rolls back space setup and returns `503 Service Unavailable`.

---

### 2. `GET /spaces`
Lists all spaces in which the authenticated user holds membership, ordered by creation time descending.

**Response (`200 OK`)**
```json
{
  "spaces": [...]
}
```

---

### 3. `GET /spaces/:id`
Returns space metadata, categories, channels visible to the caller (`VIEW_CHANNEL` resolved per channel), roles, and resolved space-level `myPermissions`.

**Errors**
- `400 validation_error`: Invalid ObjectId `:id`
- `403 not_a_member`: Caller is not a member of the space
- `404 space_not_found`: Space does not exist

---

### 4. `PATCH /spaces/:id`
Updates space metadata. Requires `MANAGE_SPACE` permission or ownership. Emits `spaces.space.updated`.

**Request Body** (at least one field required)
| Field | Type | Constraints |
|---|---|---|
| `name` | `string` | 2–100 chars |
| `description` | `string \| null` | Max 500 chars |
| `iconKey` | `string \| null` | Max 256 chars |
| `bannerKey` | `string \| null` | Max 256 chars |

**Response (`200 OK`)**: `{ "space": { ... } }`

---

### 5. `DELETE /spaces/:id`
Cascades delete of the space, roles, categories, channels, invites, members, and bans. Strictly **owner-only** (`403 only_owner`). Emits `spaces.space.deleted`.

**Response (`204 No Content`)**
