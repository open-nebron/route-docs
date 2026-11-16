# Health Check (`/health`)

Readiness probe verifying database and downstream service dependencies.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/health`
- **Authentication**: None (Public)
- **Service**: `core-api` (Port: `4002`)

---

## Response

### 200 OK — Healthy
All critical dependencies (MongoDB, Redis, and JWKS cache) are operational.

```json
{
  "status": "ok",
  "mongo": "ok",
  "redis": "ok",
  "jwks": "ok"
}
```

### 503 Service Unavailable — Degraded
One or more dependencies failed their health ping.

```json
{
  "status": "degraded",
  "mongo": "ok",
  "redis": "down",
  "jwks": "ok"
}
```

---

## Curl Example

```bash
curl -i http://localhost:4002/health
```
