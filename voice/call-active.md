# Get Active Call (`GET /calls/active`)

Retrieves the calling user's currently active voice call (`state IN ('ringing', 'accepted')`), or returns `204 No Content` if no active call exists.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/calls/active`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 120 requests per minute per authenticated user
- **Service**: `voice-service` (Port: `4004`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

---

## Behavior & Storage Operations

1. Queries ScyllaDB for active calls where `caller_id = req.user.userId` or `callee_id = req.user.userId` with `state` in `('ringing', 'accepted')`.
2. If found, returns `200 OK` with the active call object.
3. If not found, returns `204 No Content` with an empty response body.

---

## Response

### 200 OK
Active call in progress.

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

### 204 No Content
No active call currently in progress for the user. Empty body.

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `401 Unauthorized` | `invalid_token` | Missing or invalid token |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 120 requests / minute |
