# Space Invites, Membership & Moderation Bans

Endpoints for inviting users, joining spaces via invite codes, listing/kicking members, leaving spaces, and banning/unbanning members.

---

## Invite Endpoints

### `POST /spaces/:id/invites`
Creates a shareable invite code. Requires `CREATE_INVITES` permission.
- **Body**: `{ "maxUses"?: number | null (1–1000), "expiresInHours"?: number | null (1–720) }`
- **Response (`201 Created`)**: `{ "invite": { "code", "spaceId", "createdBy", "maxUses", "uses", "expiresAt", "revokedAt", "createdAt" } }`

### `GET /spaces/:id/invites`
Lists all invites for the space. Requires `MANAGE_SPACE` permission.
- **Response (`200 OK`)**: `{ "invites": [...] }`

### `DELETE /spaces/:id/invites/:code`
Revokes an active invite code. Requires `CREATE_INVITES` permission.
- **Response (`204 No Content`)** | `404 invite_not_found`

### `POST /spaces/join/:code`
Joins the space associated with `:code`.
- Checks: existence (`404 invite_not_found`), revocation (`410 invite_revoked`), expiry (`410 invite_expired`), active space ban (`403 space_banned`).
- If already a member, returns `200 OK` `{ "space": { ... }, "alreadyMember": true }` without consuming an invite use.
- Otherwise atomically increments `uses` enforcing `maxUses` (`410 invite_exhausted`), adds membership, increments `memberCount`, emits `spaces.member.added`, and returns `200 OK` `{ "space": { ... }, "alreadyMember": false }`.

---

## Member Endpoints

### `GET /spaces/:id/members?limit=50&before=<ISO>`
Cursor-paginated list of members in the space (requires membership).
- **Query**: `limit` (1–100, default 50), `before` (ISO timestamp cursor)
- **Response (`200 OK`)**: `{ "members": [{ "userId", "spaceId", "roles", "nickname", "joinedAt", "timeoutUntil", "isOwner" }], "nextCursor": "<ISO> | null" }`

### `DELETE /spaces/:id/members/:userId`
Removes a member from the space:
- **Self-leave** (`userId === caller`): Allowed for non-owners (`owner_cannot_leave` -> `400`).
- **Kick** (`userId !== caller`): Requires `KICK_MEMBERS`, cannot kick owner (`403 cannot_kick_owner`), and non-owners cannot kick members with equal or higher highest role (`403 insufficient_role_hierarchy`).
- Decrements `memberCount`, emits `spaces.member.removed`, returns `204 No Content`.

---

## Ban Endpoints

### `POST /spaces/:id/bans`
Bans a user from the space and removes any existing membership. Requires `BAN_MEMBERS` + hierarchy checks (forbids self-ban `400`, banning owner `403 cannot_ban_owner`, or banning equal/higher ranked members).
- **Body**: `{ "userId": "<ObjectId|UUID>", "reason"?: string | null (<= 500) }`
- **Response (`201 Created`)**: `{ "banned": true, "userId": "..." }`
- Emits `spaces.member.banned`.

### `GET /spaces/:id/bans`
Lists active bans in the space. Requires `BAN_MEMBERS`.
- **Response (`200 OK`)**: `{ "bans": [...] }`

### `DELETE /spaces/:id/bans/:userId`
Removes a ban so the user may re-join via invite. Requires `BAN_MEMBERS`.
- **Response (`204 No Content`)**
