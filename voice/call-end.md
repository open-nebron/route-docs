# End or Cancel Voice Call (`POST /calls/:id/end`)

Ends an active call or cancels an outbound ringing call. Callable by either participant (caller or callee).

---

## Route Overview

- **Method**: `POST`
- **Path**: `/calls/:id/end`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 30 requests per minute per authenticated user
- **Service**: `voice-service` (Port: `4004`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |
| `Content-Type` | `application/json` | No | Specifies request payload format |

### Path Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | `string` | Yes | Valid UUIDv4 of the call |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `reason` | `string` | No | Max 80 chars, default `'hangup'` | Termination reason code |

### Example Request Body
```json
{
  "reason": "hangup"
}
```

---

## Behavior & Storage Operations

1. **State Machine Verification**:
   - Loads call by ID; returns `404 Not Found` if missing.
   - Asserts `req.user.userId === callerId || req.user.userId === calleeId`; returns `403 Forbidden` (`not_participant`) otherwise.
2. **Branching Transitions**:
   - **If state === 'ringing' and actor is caller**:
     - Transitions to `state = 'cancelled'`, `endedBy = callerId`, `endReason = 'cancelled'`.
     - Publishes `call.cancelled` signal to callee.
     - Emits `voice.call.cancelled` domain event.
   - **If state === 'ringing' and actor is callee**:
     - Equivalent to reject: `state = 'rejected'`, `endedBy = calleeId`, `endReason = reason`.
     - Publishes `call.rejected` signal to caller.
     - Emits `voice.call.rejected` domain event.
   - **If state === 'accepted'**:
     - Transitions to `state = 'ended'`, `endedBy = userId`, `endReason = reason`.
     - Publishes `call.ended` signal to the other party.
     - Emits `voice.call.ended` domain event.
   - **Any other state (already ended/rejected/cancelled/missed)**:
     - Returns `409 Conflict` (`invalid_state_transition`).
3. **Database State Update**:
   - Updates `calls` table with new state and timestamps.

---

## Response

### 200 OK
Call terminated successfully.

```json
{
  "id": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "callerId": "11111111-1111-4111-8111-111111111111",
  "calleeId": "22222222-2222-4222-8222-222222222222",
  "state": "ended",
  "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "createdAt": "2026-11-22T16:30:00.000Z",
  "acceptedAt": "2026-11-22T16:30:05.000Z",
  "endedAt": "2026-11-22T16:35:10.000Z",
  "endedBy": "11111111-1111-4111-8111-111111111111",
  "endReason": "hangup"
}
```

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `401 Unauthorized` | `invalid_token` | Missing or invalid token |
| `403 Forbidden` | `not_participant` | User is not caller or callee |
| `404 Not Found` | `call_not_found` | Call ID does not exist |
| `409 Conflict` | `invalid_state_transition` | Call is already terminated |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 30 requests / minute |
