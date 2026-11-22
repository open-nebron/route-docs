# Refresh LiveKit Access Token (`POST /calls/:id/tokens`)

Issues a fresh LiveKit access token for a participant during an active accepted call (used for client reconnects, WebRTC ICE restarts, or token expiration recovery).

---

## Route Overview

- **Method**: `POST`
- **Path**: `/calls/:id/tokens`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 10 requests per minute per authenticated user
- **Service**: `voice-service` (Port: `4004`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Path Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | `string` | Yes | Valid UUIDv4 of the call |

---

## Behavior & Storage Operations

1. **Participation & State Check**:
   - Loads call by ID; returns `404 Not Found` if missing.
   - Verifies `req.user.userId === callerId || req.user.userId === calleeId`; returns `403 Forbidden` (`not_participant`) otherwise.
   - Asserts `state === 'accepted'`; returns `409 Conflict` (`invalid_state_transition`) if call is ringing or ended.
2. **Token Generation**:
   - Generates LiveKit access token for `req.user.userId` with grants for room `call.roomName`.
   - Returns token with `livekitUrl` from configuration.

---

## Response

### 200 OK
Fresh LiveKit token generated.

```json
{
  "token": "eyJhbGciOi...",
  "livekitUrl": "ws://localhost:7880",
  "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156"
}
```

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `401 Unauthorized` | `invalid_token` | Missing or invalid token |
| `403 Forbidden` | `not_participant` | User is not a participant in this call |
| `404 Not Found` | `call_not_found` | Call ID does not exist |
| `409 Conflict` | `invalid_state_transition` | Call is not in accepted state |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 10 requests / minute |
