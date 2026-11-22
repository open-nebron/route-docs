# Get Call Details (`GET /calls/:id`)

Retrieves call details, current state, participant identities, timestamps, and peer status for a participant in the call.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/calls/:id`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 120 requests per minute per authenticated user
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

1. **Existence & Permission Check**:
   - Queries ScyllaDB `calls` table by `id`.
   - If not found, returns `404 Not Found` (`call_not_found`).
   - Verifies `req.user.userId === callerId || req.user.userId === calleeId`. If not, returns `403 Forbidden` (`not_participant`).
2. **Peer Context Resolution**:
   - Resolves peer user ID (`calleeId` if caller, `callerId` if callee).
   - Resolves peer role and current state.

---

## Response

### 200 OK
Call details returned.

```json
{
  "id": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "callerId": "11111111-1111-4111-8111-111111111111",
  "calleeId": "22222222-2222-4222-8222-222222222222",
  "state": "accepted",
  "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156",
  "createdAt": "2026-11-22T16:30:00.000Z",
  "acceptedAt": "2026-11-22T16:30:05.000Z",
  "endedAt": null,
  "endedBy": null,
  "endReason": null,
  "peer": {
    "userId": "22222222-2222-4222-8222-222222222222",
    "role": "callee",
    "state": "accepted"
  }
}
```

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `401 Unauthorized` | `invalid_token` | Missing or invalid token |
| `403 Forbidden` | `not_participant` | User is not a participant in this call |
| `404 Not Found` | `call_not_found` | Call ID does not exist |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 120 requests / minute |
