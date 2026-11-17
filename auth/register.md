# Register User

Registers a new user account with email and password in the `auth-service` database (`nebron_auth`).

> [!NOTE]
> This endpoint creates the user record and publishes an asynchronous domain event (`auth.user.registered`). It does **not** issue authentication tokens upon registration. The user must subsequently call `POST /auth/login` to obtain an access and refresh token pair.

---

## Route Overview

- **Method**: `POST`
- **Path**: `/auth/register`
- **Authentication**: None (Public)
- **Rate Limit**: 5 requests per hour per IP

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Content-Type` | `application/json` | Yes | Specifies request payload format |

### Body Parameters
| Parameter | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `email` | `string` | Yes | Valid email format | User's email address (automatically trimmed and lowercased) |
| `password` | `string` | Yes | Min 12, max 128 chars; at least 1 letter + 1 number | User password |

### Example Request Body
```json
{
  "email": "developer@zenuxs.com",
  "password": "SecurePassword123!"
}
```

---

## Response

### 200 OK
User registered successfully.

```json
{
  "userId": "67389a0123456789abcdef01"
}
```

---

## Error Responses

### 400 Bad Request (`VALIDATION_ERROR`)
Payload failed Zod validation (e.g. invalid email format, password < 12 characters, missing letter/number, password > 128 characters).

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Password must be at least 12 characters long"
  }
}
```

### 409 Conflict (`EMAIL_ALREADY_EXISTS`)
The provided email address is already registered.

> [!TIP]
> To prevent timing enumeration attacks, the service executes a constant-time dummy Argon2id hash verification prior to returning this error.

```json
{
  "error": {
    "code": "EMAIL_ALREADY_EXISTS",
    "message": "Email is already registered"
  }
}
```

### 429 Too Many Requests (`RATE_LIMIT_EXCEEDED`)
IP has exceeded the rate limit threshold of 5 registrations per hour.

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests, please try again later"
  }
}
```

---

## Asynchronous Events Published

Publishes an event to the Redis Stream `zenuxs.events`:

- **Stream Key**: `zenuxs.events`
- **Field `type`**: `auth.user.registered`
- **Field `payload`** (JSON string):
  ```json
  {
    "userId": "67389a0123456789abcdef01",
    "email": "developer@zenuxs.com",
    "createdAt": "2026-11-17T00:00:00.000Z"
  }
  ```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4001/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "developer@zenuxs.com",
    "password": "SecurePassword123!"
  }'
```
