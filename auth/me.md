# Get Current User (`/auth/me`)

Verifies the client's RS256 Bearer access token and returns the authenticated user identity and session claims.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/auth/me`
- **Authentication**: `Bearer <accessToken>` (JWT RS256)
- **Rate Limit**: Default Fastify threshold

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Body Parameters
None.

---

## Response

### 200 OK
Token verified successfully.

```json
{
  "userId": "67389a0123456789abcdef01",
  "email": "developer@zenuxs.com",
  "sessionId": "67389a0123456789abcdef02"
}
```

---

## Error Responses

### 401 Unauthorized (`UNAUTHORIZED`)
`Authorization` header is missing or does not start with `Bearer `.

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Missing or invalid Authorization header"
  }
}
```

### 401 Unauthorized (`INVALID_TOKEN`)
The access token signature could not be verified with the active or rotated JWKS, or required claims (`sub`, `email`, `sid`) are absent.

```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Invalid or malformed token"
  }
}
```

### 401 Unauthorized (`TOKEN_EXPIRED`)
The 15-minute access token has expired. Client should invoke `POST /auth/refresh` to obtain a fresh token.

```json
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Access token has expired"
  }
}
```

---

## Curl Example

```bash
curl -i http://localhost:4001/auth/me \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6ImRldi0xIn0.eyJzdWIiOiI2NzM4OWEwMTIzNDU2Nzg5YWJjZGVmMDEiLCJlbWFpbCI6ImRldmVsb3BlckB6ZW51eHMuY29tIiwic2lkIjoiNjczODlhMDEyMzQ1Njc4OWFiY2RlZjAyIiwiaWF0IjoxNzAwMDAwMDAwLCJleHAiOjE3MDAwMDA5MDAsImlzcyI6InplbnV4cy1hdXRoIiwiYXVkIjpbImNvcmUiLCJjaGF0Iiwidm9pY2UiXX0..."
```
