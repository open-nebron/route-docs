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
| `/health` | `GET` | [Health Check](./chat/health.md) | ScyllaDB, Redis, and JWKS cache readiness probe | [health.md](./chat/health.md) |
| `/docs` | `GET` | [Swagger UI](./chat/docs.md) | Interactive OpenAPI 3.1 documentation | [docs.md](./chat/docs.md) |
