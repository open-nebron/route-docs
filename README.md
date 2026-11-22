# route-docs

API route documentation and endpoint specifications for the Nebron polyrepo ecosystem.

---

## Microservices Directory

### Authentication Service (`auth-service`)
Port: `4001` | Base URL: `http://localhost:4001`

| Route | Method | Title | Description | Documentation |
|---|---|---|---|---|
| `/auth/register` | `POST` | [Register User](./auth/register.md) | Create user account with email & password | [register.md](./auth/register.md) |
| `/auth/login` | `POST` | [User Login](./auth/login.md) | Authenticate user & issue RS256 access + refresh tokens | [login.md](./auth/login.md) |
| `/auth/refresh` | `POST` | [Refresh Access Token](./auth/refresh.md) | Rotate refresh token with reuse detection | [refresh.md](./auth/refresh.md) |
| `/auth/logout` | `POST` | [User Logout](./auth/logout.md) | Invalidate refresh token session (idempotent) | [logout.md](./auth/logout.md) |
| `/auth/me` | `GET` | [Current User Identity](./auth/me.md) | Verify Bearer access token and retrieve claims | [me.md](./auth/me.md) |
| `/.well-known/jwks.json` | `GET` | [JSON Web Key Set](./auth/jwks.md) | Public RSA keys for token verifiers | [jwks.md](./auth/jwks.md) |
| `/health` | `GET` | [Health Check](./auth/health.md) | MongoDB & Redis connectivity probe | [health.md](./auth/health.md) |
| `/docs` | `GET` | [Swagger UI](./auth/docs.md) | Interactive OpenAPI 3.1 documentation | [docs.md](./auth/docs.md) |

---

### Core API Service (`core-api`)
Port: `4002` | Base URL: `http://localhost:4002`

| Route | Method | Title | Description | Documentation |
|---|---|---|---|---|
| `/meta` | `GET` | [Platform Metadata](./profile/meta.md) | Public metadata: supported languages, themes, limits | [meta.md](./profile/meta.md) |
| `/username/check` | `GET` | [Check Username Availability](./profile/check-username.md) | Public check for username availability (20/min per IP) | [check-username.md](./profile/check-username.md) |
| `/me` | `GET` | [Get Current Profile](./profile/get-profile.md) | Retrieve authenticated user profile with computed media URLs | [get-profile.md](./profile/get-profile.md) |
| `/me` | `PATCH` | [Update Profile & Preferences](./profile/update-profile.md) | Update profile fields (`name`, `dateOfBirth`, `about`, `bio`, `language`, `theme`) | [update-profile.md](./profile/update-profile.md) |
| `/me/username` | `PATCH` | [Claim Username](./profile/claim-username.md) | Claim or update username for authenticated user | [claim-username.md](./profile/claim-username.md) |
| `/me/avatar/upload-url` | `POST` | [Avatar Upload Slot](./profile/media-upload-url.md) | Request presigned POST slot for direct avatar upload ($\le 5$ MB) | [media-upload-url.md](./profile/media-upload-url.md) |
| `/me/avatar/confirm` | `POST` | [Confirm Avatar Upload](./profile/media-confirm.md) | Verify upload in R2, save avatarKey, delete previous avatar | [media-confirm.md](./profile/media-confirm.md) |
| `/me/avatar` | `DELETE` | [Remove Avatar](./profile/media-delete.md) | Clear avatarKey and async delete file from storage | [media-delete.md](./profile/media-delete.md) |
| `/me/banner/upload-url` | `POST` | [Banner Upload Slot](./profile/media-upload-url.md) | Request presigned POST slot for direct banner upload ($\le 10$ MB) | [media-upload-url.md](./profile/media-upload-url.md) |
| `/me/banner/confirm` | `POST` | [Confirm Banner Upload](./profile/media-confirm.md) | Verify upload in R2, save bannerKey, delete previous banner | [media-confirm.md](./profile/media-confirm.md) |
| `/me/banner` | `DELETE` | [Remove Banner](./profile/media-delete.md) | Clear bannerKey and async delete file from storage | [media-delete.md](./profile/media-delete.md) |
| `/relationships/:userId/block` | `POST` | [Block User](./relationships/block-user.md) | Block a user and emit event (30/min per user) | [block-user.md](./relationships/block-user.md) |
| `/relationships/:userId/block` | `DELETE` | [Unblock User](./relationships/unblock-user.md) | Unblock a user and emit event (30/min per user) | [unblock-user.md](./relationships/unblock-user.md) |
| `/relationships/:userId/mute` | `POST` | [Mute User](./relationships/mute-user.md) | Mute calls and/or messages from a user (30/min per user) | [mute-user.md](./relationships/mute-user.md) |
| `/relationships/:userId/mute` | `DELETE` | [Unmute User](./relationships/unmute-user.md) | Clear all mute restrictions from a user (30/min per user) | [unmute-user.md](./relationships/unmute-user.md) |
| `/relationships/:userId` | `GET` | [Get Relationship](./relationships/get-relationship.md) | Get block and mute status with a user (300/min per user) | [get-relationship.md](./relationships/get-relationship.md) |
| `/relationships/check` | `GET` | [Batch Check Relationships](./relationships/check-relationships.md) | Internal batch check with token and cache header (600/min) | [check-relationships.md](./relationships/check-relationships.md) |
| `/health` | `GET` | [Health Check](./profile/health.md) | MongoDB, Redis, and JWKS cache readiness probe | [health.md](./profile/health.md) |
| `/internal/health` | `GET` | [Internal Health](./profile/internal-health.md) | JWKS cache age and debugging telemetry | [internal-health.md](./profile/internal-health.md) |
| `/docs` | `GET` | [Swagger UI](./profile/docs.md) | Interactive OpenAPI 3.1 documentation | [docs.md](./profile/docs.md) |

---

### Chat Service (`chat-service`)
Port: `4003` | Base URL: `http://localhost:4003`

| Route | Method | Title | Description | Documentation |
|---|---|---|---|---|
| `/conversations` | `POST` | [Start Direct Conversation](./chat/start-conversation.md) | Compute deterministic UUIDv5 conversation and seed participant records | [start-conversation.md](./chat/start-conversation.md) |
| `/conversations` | `GET` | [List Conversations](./chat/list-conversations.md) | Retrieve user conversations ordered DESC by last message timestamp with cursor | [list-conversations.md](./chat/list-conversations.md) |
| `/conversations/:id/messages` | `GET` | [List Messages](./chat/list-messages.md) | Paginated messages in reverse chronological order via TimeUUID cursor | [list-messages.md](./chat/list-messages.md) |
| `/conversations/:id/messages` | `POST` | [Send Message](./chat/send-message.md) | Post new message (1–4000 chars) and atomically update unread counts | [send-message.md](./chat/send-message.md) |
| `/conversations/:id/read` | `POST` | [Mark Read](./chat/mark-read.md) | Update read receipt up to messageId and reset unread counter | [mark-read.md](./chat/mark-read.md) |
| `/conversations/:id/messages/:messageId` | `DELETE` | [Soft-Delete Message](./chat/delete-message.md) | Soft-delete message (sender-only check) | [delete-message.md](./chat/delete-message.md) |
| `/ws` | `GET` (WS) | [WebSocket Messaging Protocol](./chat/ws-protocol.md) | Bidirectional real-time messaging, typing indicators, and presence updates | [ws-protocol.md](./chat/ws-protocol.md) |
| `/ws` (Errors) | `WS` | [WebSocket Close & Error Codes](./chat/ws-errors.md) | Specification of close codes (4401, 4403, 4408, 4409) and error frames | [ws-errors.md](./chat/ws-errors.md) |
| `/health` | `GET` | [Health Check](./chat/health.md) | ScyllaDB, Redis, and JWKS cache readiness probe | [health.md](./chat/health.md) |
| `/docs` | `GET` | [Swagger UI](./chat/docs.md) | Interactive OpenAPI 3.1 documentation | [docs.md](./chat/docs.md) |

---

### Voice Service (`voice-service`)
Port: `4004` | Base URL: `http://localhost:4004`

| Route | Method | Title | Description | Documentation |
|---|---|---|---|---|
| `/calls` | `POST` | [Initiate Call](./voice/call-start.md) | Start 1:1 call, check block/mute, push signaling events | [call-start.md](./voice/call-start.md) |
| `/calls/:id/accept` | `POST` | [Accept Call](./voice/call-accept.md) | Accept ringing call, mint LiveKit tokens for both parties | [call-accept.md](./voice/call-accept.md) |
| `/calls/:id/reject` | `POST` | [Reject Call](./voice/call-reject.md) | Reject ringing call with reason code (callee only) | [call-reject.md](./voice/call-reject.md) |
| `/calls/:id/end` | `POST` | [End / Cancel Call](./voice/call-end.md) | Terminate active call or cancel outbound ringing call | [call-end.md](./voice/call-end.md) |
| `/calls/active` | `GET` | [Get Active Call](./voice/call-active.md) | Retrieve current active ringing/accepted call or 204 | [call-active.md](./voice/call-active.md) |
| `/calls/:id` | `GET` | [Get Call Details](./voice/call-get.md) | Fetch call metadata and peer status (participant only) | [call-get.md](./voice/call-get.md) |
| `/calls/:id/tokens` | `POST` | [Refresh Tokens](./voice/call-tokens.md) | Mint fresh LiveKit token for reconnects during accepted call | [call-tokens.md](./voice/call-tokens.md) |
| `/calls` | `GET` | [List Call History](./voice/call-history.md) | Paginated past call history for user ordered DESC | [call-history.md](./voice/call-history.md) |
| `/webhooks/livekit` | `POST` | [LiveKit Webhooks](./voice/webhooks-livekit.md) | Ingest LiveKit server events (`room_finished`, `participant_left`) and auto-end calls | [webhooks-livekit.md](./voice/webhooks-livekit.md) |
| `/calls/:id/debug` | `GET` | [Call Diagnostic Debug](./voice/call-debug.md) | Internal inspection comparing ScyllaDB and LiveKit room states for divergence | [call-debug.md](./voice/call-debug.md) |
| `CallReconcileJob` | `Job` | [Call Reconciliation](./voice/call-reconcile.md) | Background self-healing job auto-ending stuck calls using Scylla LWT | [call-reconcile.md](./voice/call-reconcile.md) |
| `ws:signal:{userId}` | `Pub/Sub` | [Voice Signaling Protocol](./voice/ws-signal.md) | Specification of Redis pub/sub push frames and chat-service forwarder | [ws-signal.md](./voice/ws-signal.md) |
| Call Lifecycle | `Guide` | [End-to-End Call Lifecycle](./voice/call-flow.md) | State machine transitions, sequence diagram, and timeout sweeps | [call-flow.md](./voice/call-flow.md) |
| `/health` | `GET` | System Health | Scylla, Redis, JWKS, and LiveKit readiness check | OpenAPI |
| `/docs` | `GET` | Swagger UI | Interactive OpenAPI 3.1 documentation | OpenAPI |

