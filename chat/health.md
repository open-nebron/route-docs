# Health Check (`GET /health`)

Monitors the operational health and readiness of `chat-service` and its underlying dependencies (ScyllaDB, Redis, and JWKS cache). Used by Docker healthchecks, Kubernetes liveness/readiness probes, and load balancers.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/health`
- **Authentication**: None (Public)
- **Rate Limit**: None
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Headers
None required.

### Query Parameters
None.

### Body Parameters
None.

---

## Subsystem Checks

1. **ScyllaDB Probe**: Executes query `SELECT now() FROM system.local` with a strict 2000 ms timeout. Returns `"ok"` if successful, `"down"` on connection failure or query timeout.
2. **Redis Probe**: Issues `redis.ping()`. Expects `"PONG"`. Returns `"ok"` on success, `"down"` on connection timeout or failure.
3. **JWKS Cache Probe**: Reports the status of the in-memory cached JWKS public keys fetched from `auth-service` during startup (`"ok"` or `"stale"`).

---

## Response

### 200 OK
All connected databases and caches are operational.

```json
{
  "status": "ok",
  "scylla": "ok",
  "redis": "ok",
  "jwks": "ok",
  "version": "1.0.0"
}
```

### 503 Service Unavailable
One or more critical subsystems (ScyllaDB or Redis) failed the health ping.

```json
{
  "status": "down",
  "scylla": "down",
  "redis": "ok",
  "jwks": "ok",
  "version": "1.0.0"
}
```

---

## Curl Example

```bash
curl -i http://localhost:4003/health
```
