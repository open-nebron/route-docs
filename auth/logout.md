# User Logout

Logs out a user by revoking the session associated with the provided refresh token.

> [!NOTE]
> This endpoint is completely **idempotent**. It will always return `204 No Content` even if the provided token does not exist or has already been revoked.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/auth/logout`
- **Authentication**: None (Requires refresh token in body)
- **Rate Limit**: Default Fastify threshold

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Content-Type` | `application/json` | Yes | Specifies request payload format |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `refreshToken` | `string` | Yes | Non-empty string | The refresh token identifying the session to be revoked |

### Example Request Body
```json
{
  "refreshToken": "8f7e6d5c4b3a201918273645a4b3c2d1e0f9a8b7c6d5e4f302192a3b4c5d6e7f"
}
```

---

## Response

### 204 No Content
Session successfully revoked or was already revoked/not found. No response body is returned.

---

## Error Responses

### 400 Bad Request (`VALIDATION_ERROR`)
Refresh token was missing or empty in the request body.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Refresh token cannot be empty"
  }
}
```

---

## Asynchronous Events Published

When an active session is revoked:

- **Stream Key**: `zenuxs.events`
- **Field `type`**: `auth.session.revoked`
- **Field `payload`** (JSON string):
  ```json
  {
    "userId": "67389a0123456789abcdef01",
    "sessionId": "67389a0123456789abcdef02",
    "reason": "user_logout"
  }
  ```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4001/auth/logout \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "8f7e6d5c4b3a201918273645a4b3c2d1e0f9a8b7c6d5e4f302192a3b4c5d6e7f"
  }'
```
