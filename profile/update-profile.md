# Update Profile (`/me`)

Updates one or more profile fields (`name`, `dateOfBirth`, `bio`) for the authenticated user.

---

## Route Overview

- **Method**: `PATCH`
- **Path**: `/me`
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
All fields are optional, but **at least one field must be provided**.

| Field | Type | Constraints | Description |
|---|---|---|---|
| `name` | string | 1–80 chars | User's full or display name (control chars and newlines stripped) |
| `dateOfBirth` | string | Format `YYYY-MM-DD` | ISO date string implying age $\ge 13$ years old |
| `bio` | string | $\le 300$ chars | Short user bio/tagline (ASCII control characters stripped) |

Example Body:
```json
{
  "name": "Alexander Smith",
  "dateOfBirth": "1998-05-20",
  "bio": "Distributed systems engineer."
}
```

---

## Response

### 200 OK
Profile updated successfully.

```json
{
  "userId": "usr_9988776655",
  "username": "alex.smith",
  "name": "Alexander Smith",
  "dateOfBirth": "1998-05-20",
  "bio": "Distributed systems engineer.",
  "profilePicUrl": null,
  "createdAt": "2026-11-16T22:30:00.000Z",
  "updatedAt": "2026-11-16T23:15:00.000Z"
}
```

---

## Error Responses

### 400 Bad Request (`validation_error`)
Empty body, malformed date string, or date of birth implies age under 13.

```json
{
  "error": {
    "code": "validation_error",
    "message": "At least one field (name, dateOfBirth, bio) must be provided"
  }
}
```

```json
{
  "error": {
    "code": "validation_error",
    "message": "dateOfBirth: You must be at least 13 years old to use Zenuxs"
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
The user profile document was not found.

```json
{
  "error": {
    "code": "profile_not_found",
    "message": "Profile not found"
  }
}
```

---

## Curl Example

```bash
curl -i -X PATCH http://localhost:4002/me \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Alexander Smith",
    "dateOfBirth": "1998-05-20",
    "bio": "Distributed systems engineer."
  }'
```
