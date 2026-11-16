# Health Check

Monitors the health and readiness of `auth-service` and its underlying dependencies (MongoDB 7 and Redis 7). Used by Docker healthchecks, Kubernetes liveness/readiness probes, and load balancers.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/health`
- **Authentication**: None (Public)

---

## Request

### Headers
None required.

### Body Parameters
None.

---

## Response

### 200 OK
All connected services and internal subsystems are operational.

```json
{
  "status": "ok",
  "db": "ok",
  "redis": "ok"
}
```

### 503 Service Unavailable
One or more subsystems (MongoDB or Redis) failed their health probe ping.

```json
{
  "status": "error",
  "db": "ok",
  "redis": "error"
}
```

---

## Curl Example

```bash
curl -i http://localhost:4001/health
```
