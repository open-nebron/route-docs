# Roles & Permission Resolution

Bitfield role management and effective permission evaluation backed by `@nebron/permissions`.

---

## Permission Bits (`BigInt` serialized as decimal string)

| Bit | Flag | Value | Scope |
|---|---|---|---|
| `0` | `VIEW_CHANNEL` / `VIEW_CHANNELS` | `1` | Space & Channel |
| `1` | `MANAGE_CHANNELS` | `2` | Space & Channel |
| `2` | `MANAGE_ROLES` | `4` | Space & Channel |
| `3` | `MANAGE_SPACE` | `8` | Space |
| `4` | `KICK_MEMBERS` | `16` | Space |
| `5` | `BAN_MEMBERS` | `32` | Space |
| `6` | `CREATE_INVITES` | `64` | Space & Channel |
| `7` | `ADMINISTRATOR` | `128` | Space (all bits + override bypass) |
| `8` | `SEND_MESSAGES` | `256` | Text Channel |
| `9` | `MANAGE_MESSAGES` | `512` | Text Channel |
| `10` | `EMBED_LINKS` | `1024` | Text Channel |
| `11` | `ATTACH_FILES` | `2048` | Text Channel |
| `12` | `READ_HISTORY` | `4096` | Text Channel |
| `13` | `MENTION_EVERYONE` | `8192` | Text Channel |
| `14` | `ADD_REACTIONS` | `16384` | Text Channel |
| `15` | `CONNECT` | `32768` | Voice Channel |
| `16` | `SPEAK` | `65536` | Voice Channel |
| `17` | `VIDEO` | `131072` | Voice Channel |
| `18` | `MUTE_MEMBERS` | `262144` | Voice Channel |
| `19` | `DEAFEN_MEMBERS` | `524288` | Voice Channel |

### Evaluation Order (`resolvePermissions`)
1. Owners receive `ALL_PERMISSIONS`.
2. Base permissions = `@everyone` role OR all assigned member role permissions.
3. If `ADMINISTRATOR` bit is set, immediately returns `ALL_PERMISSIONS`.
4. Channel overrides apply sequentially:
   - `@everyone` channel override (`deny`, then `allow`)
   - Combined `allow` and combined `deny` of member's assigned role overrides (`deny` applied, then `allow`)
   - Explicit member-specific channel override (`deny`, then `allow`)

---

## Role Endpoints

- **`GET /spaces/:id/roles`** (member): Returns `{ "roles": [...] }` sorted by `position` DESC.
- **`POST /spaces/:id/roles`** (`MANAGE_ROLES`): Creates custom role (`{ "name", "color?", "permissions?", "position?" }`). Non-owners/non-admins cannot grant `ADMINISTRATOR` or set `position >= actorHighestRole`. Emits `spaces.role.changed`. Returns `201`.
- **`PATCH /spaces/:id/roles/:roleId`** (`MANAGE_ROLES`): Edits role fields (`name` on `@everyone` ignored; `@everyone` position must stay `0`). Emits `spaces.role.changed`. Returns `200`.
- **`DELETE /spaces/:id/roles/:roleId`** (`MANAGE_ROLES`): Deletes role and strips it from all members. Rejects `@everyone` (`400 cannot_delete_default_role`). Emits `spaces.role.changed`. Returns `204`.
- **`PUT /spaces/:id/members/:userId/roles`** (`MANAGE_ROLES` + hierarchy): Body `{ "roleIds": ["..."] }` (or `roles`). Strips `@everyone` automatically and enforces role hierarchy. Emits `spaces.member.roles.changed`. Returns `200 { "member": { ... } }`.

---

## Permission Evaluation Endpoints

Accepts either `X-Internal-Token` (service-to-service from `chat-service`/`voice-service`) or authenticated space member `Bearer` JWT:

### `GET /spaces/:id/permissions/:userId`
Returns space-level resolved permissions:
```json
{
  "userId": "...",
  "spaceId": "...",
  "isMember": true,
  "isOwner": false,
  "permissions": "244801"
}
```

### `GET /spaces/:id/channels/:chId/permissions/:userId`
Returns channel-level resolved permissions including channel overrides:
```json
{
  "userId": "...",
  "spaceId": "...",
  "channelId": "...",
  "isMember": true,
  "isOwner": false,
  "permissions": "244801"
}
```
