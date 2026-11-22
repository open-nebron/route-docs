# Accept Voice Call (`POST /calls/:id/accept`)

Accepts an incoming ringing call (callee only), transitions the call state to `'accepted'`, generates LiveKit access tokens for both parties, and sends real-time connected signals.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/calls/:id/accept`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 20 requests per minute per authenticated user
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

1. **State Machine Verification**:
   - Loads call by ID from ScyllaDB; returns `404 Not Found` if missing.
   - Asserts `state === 'ringing'`; returns `409 Conflict` (`invalid_state_transition`) otherwise.
   - Asserts `req.user.userId === calleeId`; returns `403 Forbidden` (`not_callee`) if caller tries to accept.
2. **Re-Check Block**:
   - Queries `core-backend` to ensure caller hasn't blocked callee since ringing began; returns `403 Forbidden` (`blocked`) if blocked.
3. **Database State Update**:
   - Updates call in ScyllaDB: `state = 'accepted'`, `accepted_at = now()`.
4. **LiveKit Token Issuance**:
   - Generates LiveKit access token for caller with room join/publish/subscribe permissions.
   - Generates LiveKit access token for callee with room join/publish/subscribe permissions.
5. **Real-Time Signaling Push**:
   - Publishes to Redis channel `ws:signal:{callerId}`:
     ```json
     {
       "type": "call.accepted",
       "payload": {
         "callId": "<id>",
         "callerToken": "<token>",
         "livekitUrl": "<LIVEKIT_URL>",
         "roomName": "<id>"
       }
     }
     ```
   - Publishes to Redis channel `ws:signal:{calleeId}`:
     ```json
     {
       "type": "call.connected",
       "payload": {
         "callId": "<id>",
         "calleeToken": "<token>",
         "livekitUrl": "<LIVEKIT_URL>",
         "roomName": "<id>"
       }
     }
     ```
6. **Domain Event Publication**:
   - Emits `voice.call.accepted` to Redis Stream `nebron.events`.

---

## Response

### 200 OK
Call accepted, room credentials returned.

```json
{
  "call": {
    "id": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "callerId": "11111111-1111-4111-8111-111111111111",
    "calleeId": "22222222-2222-4222-8222-222222222222",
    "state": "accepted",
    "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "createdAt": "2026-11-22T16:30:00.000Z",
    "acceptedAt": "2026-11-22T16:30:05.000Z"
  },
  "callerToken": "eyJhbGciOi...",
  "calleeToken": "eyJhbGciOi...",
  "livekitUrl": "ws://localhost:7880"
}
```

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `401 Unauthorized` | `invalid_token` | Missing or invalid token |
| `403 Forbidden` | `not_callee` | Authenticated user is not the callee |
| `403 Forbidden` | `blocked` | Caller has blocked callee |
| `404 Not Found` | `call_not_found` | Call ID does not exist |
| `409 Conflict` | `invalid_state_transition` | Call is not in ringing state |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 20 requests / minute |
| `503 Service Unavailable` | `service_unavailable` | Core relationship service unavailable |
