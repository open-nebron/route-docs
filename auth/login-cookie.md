# User Login (Cookie-Based for Web Clients)

Authenticates a user with email and password, creates a persistent session record in MongoDB, issues an RS256 JWT access token in the response body, and sets a secure `HttpOnly` cookie containing the opaque refresh token (`refresh_token`).

> [!NOTE]
> This endpoint is designed for browser-based web frontend clients to protect refresh tokens against Cross-Site Scripting (XSS). Native and mobile clients should use `POST /auth/login`.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/auth/login/cookie`
- **Authentication**: None (Public)
- **Rate Limit**: 
  - **10 requests per minute per IP**
  - **5 requests per minute per email**
  - *(Whichever threshold is reached first triggers HTTP 429)*
- **CORS**: Requires explicit origin match from `ALLOWED_ORIGINS` and `credentials: true`. Preflight `OPTIONS` returns `Access-Control-Allow-Credentials: true`.

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Content-Type` | `application/json` | Yes | Specifies request payload format |
| `Origin` | `string` | Yes (in browser) | Client origin domain for CORS credentials |
| `User-Agent` | `string` | No | Client user agent string (stored in session record) |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `email` | `string` | Yes | Valid email format | User's registered email |
| `password` | `string` | Yes | Max 128 characters | User password |

### Example Request Body
```json
{
  "email": "developer@nebron.com",
  "password": "SecurePassword123!"
}
```

---

## Response

### 200 OK
Authentication successful. Returns access token and expiration in seconds in the response body, and sets the `refresh_token` HttpOnly cookie.

#### Response Headers
```http
Set-Cookie: refresh_token=e859669752a90328a7f996c804f08a7b29fb1813f61474ad5d12a6a13c6f612a; Max-Age=2592000; Path=/auth; HttpOnly; SameSite=Lax
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Credentials: true
```

#### Response Body
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6ImRldi0xIn0.eyJzdWIiOiI2NzM4OWEwMTIzNDU2Nzg5YWJjZGVmMDEiLCJlbWFpbCI6ImRldmVsb3BlckBuZWJyb24uY29tIiwic2lkIjoiNjczODlhMDEyMzQ1Njc4OWFiY2RlZjAyIiwiaWF0IjoxNzAwMDAwMDAwLCJleHAiOjE3MDAwMDA5MDAsImlzcyI6Im5lYnJvbi1hdXRoIiwiYXVkIjpbImNvcmUiLCJjaGF0Iiwidm9pY2UiXX0...",
  "expiresIn": 900
}
```

> [!IMPORTANT]
> The response body does **NOT** contain `refreshToken`. The refresh token is transmitted exclusively through the `Set-Cookie` header.

#### Cookie Attributes
- **Name**: `refresh_token`
- **HttpOnly**: `true` (Cannot be read by `document.cookie`)
- **Secure**: `true` in production (`false` in local development)
- **SameSite**: `Lax` (configurable to `None` for cross-subdomain architectures)
- **Path**: `/auth`
- **Max-Age**: `2592000` (30 days in seconds)

---

## Error Responses

### 400 Bad Request (`VALIDATION_ERROR`)
Request validation failed.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format"
  }
}
```

### 401 Unauthorized (`INVALID_CREDENTIALS` / `UNAUTHORIZED`)
Provided credentials do not match or user account is disabled. No `Set-Cookie` header is returned.

```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password"
  }
}
```

### 429 Too Many Requests (`RATE_LIMIT_EXCEEDED`)
IP or email rate limit exceeded.

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests, please try again later"
  }
}
```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4001/auth/login/cookie \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d '{
    "email": "developer@nebron.com",
    "password": "SecurePassword123!"
  }'
```
