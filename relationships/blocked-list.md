# List Blocked Users (`GET /relationships/blocked`)

Returns the list of users blocked by the authenticated user, enriched with profile information (username, name, avatar URL).

---

## Route Overview

- **Method**: `GET`
- **Path**: `/relationships/blocked`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"core"`)
- **Rate Limit**: 60 requests per minute per user
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Path Parameters
None.

### Query Parameters
None.

---

## Response

### 200 OK
Returns the array of blocked user objects.

```json
{
  "blocked": [
    {
      "userId": "6b055f7d0e785a20e6ca6533",
      "blockedAt": "2026-11-24T12:00:00.000Z",
      "username": "spammer_user",
      "name": "Spammer User",
      "avatarUrl": "https://cdn.nebron.com/avatars/user.jpg"
    }
  ]
}
```

### 401 Unauthorized
Missing, expired, or invalid bearer token.
