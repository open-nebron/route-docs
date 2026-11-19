# User Login

Authenticates a user with email and password, creates a persistent session record in MongoDB, and issues an RS256 JWT access token along with an opaque 32-byte refresh token.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/auth/login`
- **Authentication**: None (Public)
- **Rate Limit**: 
  - **10 requests per minute per IP**
  - **5 requests per minute per email**
  - *(Whichever threshold is reached first triggers HTTP 429)*

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Content-Type` | `application/json` | Yes | Specifies request payload format |
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
Authentication successful. Returns access token, refresh token, and access token lifespan in seconds.

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6ImRldi0xIn0.eyJzdWIiOiI2NzM4OWEwMTIzNDU2Nzg5YWJjZGVmMDEiLCJlbWFpbCI6ImRldmVsb3BlckB6ZW51eHMuY29tIiwic2lkIjoiNjczODlhMDEyMzQ1Njc4OWFiY2RlZjAyIiwiaWF0IjoxNzAwMDAwMDAwLCJleHAiOjE3MDAwMDA5MDAsImlzcyI6InplbnV4cy1hdXRoIiwiYXVkIjpbImNvcmUiLCJjaGF0Iiwidm9pY2UiXX0...",
  "refreshToken": "3a7b9c1d2e3f405162738495a6b7c8d9e0f1a2b3c4d5e6f708192a3b4c5d6e7f",
  "expiresIn": 900
}
```

#### Token Details
- **Access Token**:
  - Algorithm: `RS256` (Asymmetric RSA signature)
  - Lifespan: 15 minutes (`900` seconds)
  - Claims:
    - `sub`: User ID (`userId`)
    - `email`: User email
    - `sid`: Session ID (`sessionId`)
    - `iss`: `"nebron-auth"`
    - `aud`: `["core", "chat", "voice"]`
- **Refresh Token**:
  - Format: Opaque 32-byte cryptographically secure random string (64 hex characters)
  - Storage: Only its SHA-256 hash is persisted in the database
  - Lifespan: 30 days (enforced via MongoDB TTL index)

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
Provided credentials do not match or user account is disabled.

> [!TIP]
> If the email does not exist in the database, the server runs a dummy Argon2id hash check to ensure response timing is indistinguishable from an invalid password attempt.

```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password"
  }
}
```

### 429 Too Many Requests (`RATE_LIMIT_EXCEEDED`)
Exceeded rate limits (either 10 attempts/min per IP or 5 attempts/min per email address).

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many login attempts for this email, please try again later"
  }
}
```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4001/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "developer@nebron.com",
    "password": "SecurePassword123!"
  }'
```
