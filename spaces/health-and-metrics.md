# Spaces Service Health & Metrics

Liveness, readiness, JWKS cache telemetry, and Prometheus metrics exposition for `spaces-service`.

---

## Endpoints

### `GET /health`
Probes MongoDB, Redis, and local JWKS key availability.
- **200 OK**: `{ "status": "ok", "mongo": "ok", "redis": "ok", "jwks": "ok" }`
- **503 Service Unavailable**: `{ "status": "degraded", ... }`

### `GET /internal/health`
Returns JWKS cache diagnostics (`lastFetchedAt`, key count, initialization status).
- **200 OK**: `{ "status": "ok", "jwks": { ... } }`

### `GET /metrics`
Prometheus text exposition format (`text/plain; version=0.0.4; charset=utf-8`):
- `process_uptime_seconds`
- `spaces_http_requests_total`

### `GET /docs`
Interactive OpenAPI 3.1 Swagger UI documentation.
