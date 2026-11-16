# Update Profile & Preferences (`/me`)

Updates one or more profile fields (`name`, `dateOfBirth`, `about`, `bio`, `language`, `theme`) for the authenticated user.

---

## Route Overview

- **Method**: `PATCH`
- **Path**: `/me`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"core"`)
- **Rate Limit**: 30 requests per minute per user
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token |
| `Content-Type` | `application/json` | Yes | Request body format |

### Body Parameters
All fields are optional, but **at least one field must be provided**. Unknown fields are rejected (`.strict()`).

| Field | Type | Constraints | Description |
|---|---|---|---|
| `name` | string | 1–80 chars | User's full or display name (control chars and newlines stripped) |
| `dateOfBirth` | string | Format `YYYY-MM-DD` | ISO date string implying age $\ge 13$ years old |
| `about` | string | $\le 2000$ chars, $\le 100$ newlines | Long about-me description. No HTML tags or null bytes allowed. |
| `bio` | string | $\le 300$ chars | Short tagline/bio. No HTML tags or null bytes allowed. |
| `language` | string | In `SUPPORTED_LANGUAGES` | BCP-47 lowercase language code (e.g. `en`, `hi`, `ta`, `te`, etc.) |
| `theme` | string | `"light" \| "dark" \| "system"` | User UI theme preference |

Example Body:
```json
{
  "name": "Alexander Smith",
  "dateOfBirth": "1998-05-20",
  "about": "Distributed systems engineer building high-throughput microservices.",
  "bio": "Building Zenuxs Core",
  "language": "hi",
  "theme": "dark"
}
```

---

## Response

### 200 OK
Profile updated successfully. Returns full profile shape with computed media URLs.

```json
{
  "userId": "usr_9988776655",
  "username": "alex.smith",
  "name": "Alexander Smith",
  "dateOfBirth": "1998-05-20",
  "about": "Distributed systems engineer building high-throughput microservices.",
  "bio": "Building Zenuxs Core",
  "language": "hi",
  "theme": "dark",
  "avatarUrl": "https://media.zenuxs.com/users/usr_9988776655/avatar/uuid-123.jpg?v=1763428900000",
  "bannerUrl": null,
  "createdAt": "2026-11-16T22:30:00.000Z",
  "updatedAt": "2026-11-17T00:15:00.000Z"
}
```

---

## Error Responses

### 400 Bad Request (`validation_error`)
Empty body, HTML tags in about/bio, length exceeded, or unsupported language/theme.

```json
{
  "error": {
    "code": "validation_error",
    "message": "about: HTML tags are not allowed in about"
  }
}
```

```json
{
  "error": {
    "code": "validation_error",
    "message": "language: Unsupported language 'xx'. Supported languages: en, hi, ta, te, ml, kn, bn, mr, gu, pa, ur"
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

### 429 Too Many Requests (`rate_limit_exceeded`)
Exceeded 30 requests per minute per user.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Maximum 30 requests per minute."
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
    "about": "Distributed systems engineer building high-throughput microservices.",
    "bio": "Building Zenuxs Core",
    "language": "hi",
    "theme": "dark"
  }'
```
