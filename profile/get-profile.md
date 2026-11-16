# Get Current Profile (`/me`)

Retrieves the authenticated user's profile from the `zenuxs_core` database.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/me`
- **Authentication**: `Bearer <accessToken>` (RS256 JWT, audience must include `"core"`)
- **Rate Limit**: Default Fastify threshold
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token issued by `auth-service` |

### Query Parameters
None.

### Body Parameters
None.

---

## Response

### 200 OK
Profile found and returned successfully.

```json
{
  "userId": "usr_9988776655",
  "username": "alex.smith",
  "name": "Alex Smith",
  "dateOfBirth": "1998-05-20",
  "bio": "Software engineer building decentralized tools.",
  "profilePicUrl": null,
  "createdAt": "2026-11-16T22:30:00.000Z"
}
```

---

## Error Responses

### 401 Unauthorized (`invalid_token`)
Missing, expired, untrusted signature, or invalid issuer/audience access token.

```json
{
  "error": {
    "code": "invalid_token",
    "message": "Authorization header is required"
  }
}
```

```json
{
  "error": {
    "code": "invalid_token",
    "message": "jwt expired"
  }
}
```

### 404 Not Found (`profile_not_found`)
Profile document does not exist in `zenuxs_core.profiles`. This indicates event consumer lag between registration on `auth-service` and profile creation on `core-api`.

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
curl -i http://localhost:4002/me \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```
