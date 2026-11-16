# Remove Media (`DELETE /me/avatar` & `DELETE /me/banner`)

Removes the user's avatar or banner from their profile document and triggers an asynchronous deletion of the underlying object in storage. This endpoint is idempotent.

---

## Route Overview

- **Methods**: `DELETE`
- **Paths**:
  - `/me/avatar`
  - `/me/banner`
- **Authentication**: `Bearer <accessToken>`
- **Rate Limit**: 10 requests per minute per user
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token |

### Body Parameters
None.

---

## Response

### 204 No Content
Media was cleared successfully, or was already unset (idempotent).

```http
HTTP/1.1 204 No Content
```

---

## Error Responses

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

---

## Curl Example

```bash
curl -i -X DELETE http://localhost:4002/me/avatar \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```
