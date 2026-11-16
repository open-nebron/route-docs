# route-docs

API route documentation and endpoint specifications for the Zenuxs polyrepo ecosystem.

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
