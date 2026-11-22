# User Logout (Cookie-Based for Web Clients)

Logs out a user by revoking the session associated with the `refresh_token` HttpOnly cookie and clearing the cookie on the client.

> [!NOTE]
> This endpoint is completely **idempotent**. It always returns `204 No Content` and clears the cookie even if the cookie is not present, expired, or already revoked.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/auth/logout/cookie`
- **Authentication**: None (Reads `refresh_token` from cookie if present)
- **Rate Limit**: 30 requests per minute per IP
- **CORS**: Requires explicit origin match from `ALLOWED_ORIGINS` and `credentials: true`.

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Cookie` | `refresh_token=<token>` | No | HttpOnly cookie containing the refresh token to revoke |
| `Origin` | `string` | Yes (in browser) | Client origin domain for CORS credentials |

### Body Parameters
*None required. Request body must be empty or omitted.*

---

## Response

### 204 No Content
Session successfully revoked (if cookie was present and valid) and cookie cleared. No response body is returned.

#### Response Headers
```http
Set-Cookie: refresh_token=; Max-Age=0; Path=/auth; Expires=Thu, 01 Jan 1970 00:00:00 GMT; HttpOnly; SameSite=Lax
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Credentials: true
```

---

## Error Responses

### 429 Too Many Requests (`RATE_LIMIT_EXCEEDED`)
IP has exceeded 30 logout requests per minute.

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests, please try again later"
  }
}
```

---

## Asynchronous Events Published

When an active session is revoked:

- **Stream Key**: `nebron.events`
- **Field `type`**: `auth.session.revoked`
- **Field `payload`** (JSON string):
  ```json
  {
    "userId": "67389a0123456789abcdef01",
    "sessionId": "67389a0123456789abcdef02",
    "reason": "user_logout"
  }
  ```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4001/auth/logout/cookie \
  -b cookies.txt \
  -c cookies.txt
```
