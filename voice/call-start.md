# Initiate 1:1 Voice Call (`POST /calls`)

Initiates a 1:1 voice call to a target peer user in ScyllaDB, checks relationship permissions, and publishes real-time WebSocket signals.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/calls`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 10 requests per minute per authenticated user
- **Service**: `voice-service` (Port: `4004`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |
| `Content-Type` | `application/json` | Yes | Specifies request payload format |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `calleeId` | `string` | Yes | Valid UUIDv4 format | UUID of the peer user to call |

### Example Request Body
```json
{
  "calleeId": "22222222-2222-4222-8222-222222222222"
}
```

---

## Behavior & Storage Operations

1. **Self-Call Prevention**: Rejects call if `calleeId === req.user.userId` with `400 Bad Request` (`cannot_call_self`).
2. **Relationship Enforcement**:
   - Queries `core-backend` via `GET /relationships/check?userIds=<calleeId>&fromUserId=<callerId>` with `X-Internal-Token`.
   - If caller is blocked by callee (or vice-versa), returns `403 Forbidden` (`blocked`).
   - If callee has muted calls from caller, returns `403 Forbidden` (`muted`).
   - If core service fails or times out, fails closed with `503 Service Unavailable` (`service_unavailable`).
3. **Active Call Concurrency Check**:
   - Checks if caller or callee is already participating in a call with state `'ringing'` or `'accepted'`.
   - If either party is active, returns `409 Conflict` (`already_in_call`).
4. **ScyllaDB Record Insertion**:
   - Generates `id = uuidv4()`, sets `roomName = id`, `state = 'ringing'`, `createdAt = now()`.
   - Inserts row into `calls` table.
5. **Real-Time Signaling Push**:
   - Publishes to Redis channel `ws:signal:{calleeId}`:
     ```json
     {
       "type": "call.ringing",
       "payload": {
         "callId": "<id>",
         "callerId": "<callerId>",
         "roomName": "<id>",
         "startedAt": "<ISO>"
       }
     }
     ```
   - Publishes to Redis channel `ws:signal:{callerId}`:
     ```json
     {
       "type": "call.dialing",
       "payload": {
         "callId": "<id>",
         "calleeId": "<calleeId>"
       }
     }
     ```
6. **Domain Event Publication**:
   - Emits `voice.call.ringing` to Redis Stream `nebron.events`.

---

## Response

### 201 Created
Call successfully initiated and ringing.

```json
{
  "id": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "callerId": "11111111-1111-4111-8111-111111111111",
  "calleeId": "22222222-2222-4222-8222-222222222222",
  "state": "ringing",
  "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "createdAt": "2026-11-22T16:30:00.000Z"
}
```

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `400 Bad Request` | `cannot_call_self` | Caller attempted to call their own user ID |
| `400 Bad Request` | `invalid_payload` | `calleeId` is missing or not a valid UUID |
| `401 Unauthorized` | `invalid_token` | Missing or expired JWT Bearer token |
| `403 Forbidden` | `blocked` | Caller is blocked by callee or has blocked callee |
| `403 Forbidden` | `muted` | Callee has enabled call muting for caller |
| `409 Conflict` | `already_in_call` | Either caller or callee is currently in an active call |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 10 requests / minute |
| `503 Service Unavailable` | `service_unavailable` | Dependency service unavailable |
