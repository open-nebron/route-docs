# List Call History (`GET /calls`)

Retrieves paginated history of past calls where the authenticated user was either the caller or callee, sorted descending by creation time.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/calls`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"voice"`)
- **Rate Limit**: 60 requests per minute per authenticated user
- **Service**: `voice-service` (Port: `4004`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Query Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `limit` | `integer` | No | Default `30`, min `1`, max `100` | Maximum number of calls to retrieve |
| `before` | `string` | No | ISO 8601 timestamp format | Paginate calls created prior to this timestamp |

---

## Behavior & Storage Operations

1. Queries ScyllaDB for all calls where `caller_id = userId` or `callee_id = userId`.
2. Deduplicates records across caller and callee queries.
3. Filters by `createdAt < before` if pagination cursor is specified.
4. Sorts descending by `createdAt`.
5. Returns `calls` slice and computes `nextCursor` if `calls.length === limit`.

---

## Response

### 200 OK
Call history retrieved.

```json
{
  "calls": [
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
  ],
  "nextCursor": "2026-11-22T16:30:00.000Z"
}
```

---

## Error Responses

| Status Code | Error Code | Description |
|---|---|---|
| `401 Unauthorized` | `invalid_token` | Missing or invalid token |
| `429 Too Many Requests` | `rate_limit_exceeded` | Exceeded 60 requests / minute |
