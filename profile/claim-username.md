# Claim or Change Username (`/me/username`)

Allows an authenticated user to claim or update their username.

---

## Route Overview

- **Method**: `PATCH`
- **Path**: `/me/username`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"core"`)
- **Rate Limit**: Default Fastify threshold
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token |
| `Content-Type` | `application/json` | Yes | Request body format |

### Body Parameters
| Field | Type | Required | Description |
|---|---|---|---|
| `username` | string | Yes | The desired username |

Example Body:
```json
{
  "username": "neo_prime"
}
```

---

## Response

### 200 OK
Username updated successfully.

```json
{
  "username": "neo_prime"
}
```

---

## Error Responses

### 400 Bad Request (`invalid_username`)
Username does not meet validation rules, is on the reserved list, or fails profanity checks.

```json
{
  "error": {
    "code": "invalid_username",
    "message": "Invalid username format"
  }
}
```

```json
{
  "error": {
    "code": "invalid_username",
    "message": "This username is reserved"
  }
}
```

### 401 Unauthorized (`invalid_token`)
Missing or invalid Bearer access token.

```json
{
  "error": {
    "code": "invalid_token",
    "message": "Authorization header is required"
  }
}
```

### 404 Not Found (`profile_not_found`)
Profile document does not exist for the authenticated user.

```json
{
  "error": {
    "code": "profile_not_found",
    "message": "Profile not found"
  }
}
```

### 409 Conflict (`username_taken`)
The requested username is already claimed by another user.

```json
{
  "error": {
    "code": "username_taken",
    "message": "Username is already taken"
  }
}
```

---

## Curl Example

```bash
curl -i -X PATCH http://localhost:4002/me/username \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"username": "neo_prime"}'
```
