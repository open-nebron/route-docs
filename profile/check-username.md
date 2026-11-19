# Check Username Availability (`/username/check`)

Public endpoint used on signup and onboarding screens to verify if a username is available.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/username/check`
- **Authentication**: None (Public)
- **Rate Limit**: 20 requests per minute per IP (tracked atomically in Redis)
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
None required.

### Query Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `username` | string | Yes | The username candidate to validate and check |

Example Query:
```
GET /username/check?username=neo_123
```

---

## Validation & Business Rules

1. **Length**: 3 to 20 characters
2. **Allowed Characters**: Lowercase `a-z`, digits `0-9`, underscore `_`, dot `.`
3. **Prefix Constraint**: Must start with a letter (`^[a-z]`)
4. **Suffix Constraint**: Must not end with `_` or `.`
5. **No Consecutive Punctuation**: No `__`, `..`, `._`, `_.`, or `_._`
6. **Reserved Words**: `admin, root, system, support, nebron, api, auth, chat, voice, me, null, undefined, help, about, settings, login, register`
7. **Profanity Filter**: Checked against the `bad-words` dictionary
8. **Privacy / Enumeration Protection**: Never leaks whether a username belongs to a specific user; only reports overall availability.

---

## Response

### 200 OK — Available
The username is valid, unreserved, and not taken in MongoDB.

```json
{
  "available": true
}
```

### 200 OK — Taken
The username is valid in format, but already registered to another user.

```json
{
  "available": false,
  "reason": "taken"
}
```

### 200 OK — Reserved
The username matches a system or reserved term.

```json
{
  "available": false,
  "reason": "reserved"
}
```

### 200 OK — Invalid Format
The username violates length, allowed characters, prefix/suffix, consecutive punctuation, or profanity rules.

```json
{
  "available": false,
  "reason": "invalid_format"
}
```

---

## Error Responses

### 400 Bad Request (`validation_error`)
The `username` query parameter is missing or empty.

```json
{
  "error": {
    "code": "validation_error",
    "message": "username: Username query parameter is required"
  }
}
```

### 429 Too Many Requests (`rate_limit_exceeded`)
Exceeded the rate limit of 20 requests per minute per IP.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Maximum 20 requests per minute."
  }
}
```

Headers returned on rate limit:
```http
X-RateLimit-Limit: 20
X-RateLimit-Remaining: 0
```

---

## Curl Example

```bash
# Check available
curl -i "http://localhost:4002/username/check?username=alice_smith"

# Check reserved
curl -i "http://localhost:4002/username/check?username=admin"
```
