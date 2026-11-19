# Refresh Access Token

Rotates the refresh token and issues a new access token. Employs token rotation and reuse detection to protect against token theft and replay attacks.

> [!CAUTION]
> **Reuse Detection**: If an already revoked refresh token is presented, `auth-service` treats it as a security compromise. It automatically revokes the **entire session chain** for that user and emits an `auth.session.revoked` event with `reason: "reuse_detected"`.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/auth/refresh`
- **Authentication**: None (Requires valid opaque refresh token in body)
- **Rate Limit**: 30 requests per minute per IP

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Content-Type` | `application/json` | Yes | Specifies request payload format |
| `User-Agent` | `string` | No | Client user agent string (recorded with new session) |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `refreshToken` | `string` | Yes | Non-empty hex string | The 32-byte opaque refresh token received from login or a prior refresh |

### Example Request Body
```json
{
  "refreshToken": "3a7b9c1d2e3f405162738495a6b7c8d9e0f1a2b3c4d5e6f708192a3b4c5d6e7f"
}
```

---

## Response

### 200 OK
Token rotation successful. The old refresh token is marked revoked, and a newly generated pair is returned.

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6ImRldi0xIn0.eyJzdWIiOiI2NzM4OWEwMTIzNDU2Nzg5YWJjZGVmMDEiLCJlbWFpbCI6ImRldmVsb3BlckB6ZW51eHMuY29tIiwic2lkIjoiNjczODlhMDEyMzQ1Njc4OWFiY2RlZjAzIiwiaWF0IjoxNzAwMDAwOTAwLCJleHAiOjE3MDAwMDE4MDAsImlzcyI6InplbnV4cy1hdXRoIiwiYXVkIjpbImNvcmUiLCJjaGF0Iiwidm9pY2UiXX0...",
  "refreshToken": "8f7e6d5c4b3a201918273645a4b3c2d1e0f9a8b7c6d5e4f302192a3b4c5d6e7f",
  "expiresIn": 900
}
```

---

## Error Responses

### 400 Bad Request (`VALIDATION_ERROR`)
Refresh token missing or empty in request body.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Refresh token cannot be empty"
  }
}
```

### 401 Unauthorized (`TOKEN_REVOKED` — Reuse Detected)
An attempt was made to reuse a refresh token that was already revoked in a prior refresh. All sessions in the user's session chain are immediately invalidated.

```json
{
  "error": {
    "code": "TOKEN_REVOKED",
    "message": "Refresh token has been revoked"
  }
}
```

### 401 Unauthorized (`TOKEN_EXPIRED`)
The refresh token has exceeded its 30-day validity period.

```json
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Refresh token has expired"
  }
}
```

### 401 Unauthorized (`UNAUTHORIZED`)
Invalid refresh token hash, or the user account is inactive.

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
curl -i -X POST http://localhost:4001/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "3a7b9c1d2e3f405162738495a6b7c8d9e0f1a2b3c4d5e6f708192a3b4c5d6e7f"
  }'
```
