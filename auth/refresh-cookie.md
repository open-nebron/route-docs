# Refresh Access Token (Cookie-Based for Web Clients)

Rotates the refresh token stored in the `refresh_token` HttpOnly cookie and issues a new access token in the response body. Employs token rotation, cookie rotation, and reuse detection to protect against token theft and replay attacks.

> [!CAUTION]
> **Reuse Detection**: If an already revoked refresh token cookie is presented, `auth-service` treats it as a security breach. It immediately revokes the **entire session chain** for that user, clears the client cookie (`Max-Age=0`), and emits an `auth.session.revoked` event with `reason: "reuse_detected"`.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/auth/refresh/cookie`
- **Authentication**: None (Requires valid opaque `refresh_token` in cookie)
- **Rate Limit**: 30 requests per minute per IP
- **CORS**: Requires explicit origin match from `ALLOWED_ORIGINS` and `credentials: true`.

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Cookie` | `refresh_token=<token>` | Yes | HttpOnly cookie containing the 32-byte opaque refresh token |
| `Origin` | `string` | Yes (in browser) | Client origin domain for CORS credentials |
| `User-Agent` | `string` | No | Client user agent string (recorded with rotated session) |

### Body Parameters
*None required. Request body must be empty or omitted.*

---

## Response

### 200 OK
Token rotation successful. The old refresh token is revoked in MongoDB, a new session is created, a rotated `Set-Cookie` is set, and a new access token is returned in the body.

#### Response Headers
```http
Set-Cookie: refresh_token=2fa0a1a3b7c45edafdbfd9670bfd5003e871ad1d53076440bac59f1fab83240a; Max-Age=2592000; Path=/auth; HttpOnly; SameSite=Lax
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Credentials: true
```

#### Response Body
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6ImRldi0xIn0.eyJzdWIiOiI2NzM4OWEwMTIzNDU2Nzg5YWJjZGVmMDEiLCJlbWFpbCI6ImRldmVsb3BlckBuZWJyb24uY29tIiwic2lkIjoiNjczODlhMDEyMzQ1Njc4OWFiY2RlZjAzIiwiaWF0IjoxNzAwMDAwOTAwLCJleHAiOjE3MDAwMDE4MDAsImlzcyI6Im5lYnJvbi1hdXRoIiwiYXVkIjpbImNvcmUiLCJjaGF0Iiwidm9pY2UiXX0...",
  "expiresIn": 900
}
```

---

## Error Responses

### 401 Unauthorized (`missing_refresh_token`)
The `refresh_token` cookie was missing from the request.

```json
{
  "error": {
    "code": "missing_refresh_token",
    "message": "Missing refresh token cookie"
  }
}
```

### 401 Unauthorized (`TOKEN_REVOKED` — Reuse Detected)
An attempt was made to present a refresh token cookie that was already revoked during a previous rotation. All sessions in the user chain are invalidated, and the cookie is cleared.

#### Response Headers
```http
Set-Cookie: refresh_token=; Max-Age=0; Path=/auth; HttpOnly; SameSite=Lax
```

#### Response Body
```json
{
  "error": {
    "code": "TOKEN_REVOKED",
    "message": "Refresh token has been revoked"
  }
}
```

### 401 Unauthorized (`TOKEN_EXPIRED`)
The refresh token cookie has exceeded its 30-day validity window.

```json
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Refresh token has expired"
  }
}
```

### 401 Unauthorized (`UNAUTHORIZED`)
Invalid token hash or inactive user account.

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid refresh token"
  }
}
```

### 429 Too Many Requests (`RATE_LIMIT_EXCEEDED`)
IP has exceeded 30 refresh requests per minute.

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests, please try again later"
  }
}
```

---

## Asynchronous Events Published (Upon Reuse Detection)

- **Stream Key**: `nebron.events`
- **Field `type`**: `auth.session.revoked`
- **Field `payload`** (JSON string):
  ```json
  {
    "userId": "67389a0123456789abcdef01",
    "sessionId": "67389a0123456789abcdef02",
    "reason": "reuse_detected"
  }
  ```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4001/auth/refresh/cookie \
  -b cookies.txt \
  -c cookies.txt
```
