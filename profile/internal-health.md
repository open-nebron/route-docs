# Internal Health & JWKS Debugging (`/internal/health`)

Debugging endpoint returning telemetry regarding the in-memory JWKS cache and background key refresh cycle.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/internal/health`
- **Authentication**: None (Internal / Debug)
- **Service**: `core-api` (Port: `4002`)

---

## Response

### 200 OK
JWKS status returned successfully.

```json
{
  "status": "ok",
  "jwks": {
    "isLoaded": true,
    "ageSeconds": 245,
    "lastFetchedAt": "2026-11-16T23:50:00.000Z",
    "keyCount": 1,
    "jwksUrl": "http://auth:4001/.well-known/jwks.json",
    "lastError": null
  }
}
```

---

## Curl Example

```bash
curl -i http://localhost:4002/internal/health
```
