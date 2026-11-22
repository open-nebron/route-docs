# Reject Voice Call (`POST /calls/:id/reject`)

Rejects an incoming ringing call (callee only), transitions the call state to `'rejected'`, records the rejection reason, and notifies the caller in real time.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/calls/:id/reject`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 20 requests per minute per authenticated user
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
| `reason` | `string` | No | Max 80 chars, default `'reject'` | Rejection reason code (e.g. `'busy'`, `'reject'`) |

### Example Request Body
```json
{
  "reason": "busy"
}
```

---

## Behavior & Storage Operations

1. **State Machine Verification**:
   - Loads call by ID; returns `404 Not Found` if missing.
   - Asserts `state === 'ringing'`; returns `409 Conflict` (`invalid_state_transition`) otherwise.
   - Asserts `req.user.userId === calleeId`; returns `403 Forbidden` (`not_callee`) if caller tries to reject.
2. **Database State Update**:
   - Updates call in ScyllaDB: `state = 'rejected'`, `ended_at = now()`, `ended_by = userId`, `end_reason = reason`.
3. **Real-Time Signaling Push**:
   - Publishes to Redis channel `ws:signal:{callerId}`:
     ```json
     {
       "type": "call.rejected",
       "payload": {
         "callId": "<id>",
         "calleeId": "<calleeId>",
         "reason": "<reason>"
       }
     }
     ```
4. **Domain Event Publication**:
   - Emits `voice.call.rejected` to Redis Stream `nebron.events`.

---

## Response

### 200 OK
Call successfully rejected.

```json
{
  "id": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "callerId": "11111111-1111-4111-8111-111111111111",
  "calleeId": "22222222-2222-4222-8222-222222222222",
  "state": "rejected",
  "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "createdAt": "2026-11-22T16:30:00.000Z",
  "endedAt": "2026-11-22T16:30:04.000Z",
  "endedBy": "22222222-2222-4222-8222-222222222222",
  "endReason": "busy"
}
```

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `401 Unauthorized` | `invalid_token` | Missing or invalid token |
| `403 Forbidden` | `not_callee` | Authenticated user is not the callee |
| `404 Not Found` | `call_not_found` | Call ID does not exist |
| `409 Conflict` | `invalid_state_transition` | Call is not in ringing state |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 20 requests / minute |
