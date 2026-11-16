# Confirm Media Upload (`/me/avatar/confirm` & `/me/banner/confirm`)

Confirms that the client successfully uploaded the image to storage. Verifies that the object exists in R2, verifies size and content type via S3 `HeadObject`, updates the user's `avatarKey` or `bannerKey`, and initiates an asynchronous deletion of any previous media file.

---

## Route Overview

- **Methods**: `POST`
- **Paths**:
  - `/me/avatar/confirm`
  - `/me/banner/confirm`
- **Authentication**: `Bearer <accessToken>`
- **Rate Limit**: 10 requests per minute per user
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token |
| `Content-Type` | `application/json` | Yes | Request payload format |

### Body Parameters
| Field | Type | Required | Description |
|---|---|---|---|
| `objectKey` | string | Yes | The key returned from Step 1 (`users/{userId}/{kind}/{uuid}.{ext}`) |

Example Body:
```json
{
  "objectKey": "users/usr_9988776655/avatar/3f7a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c.png"
}
```

---

## Security & Verification Checks

1. **User Prefix Check**: Asserts that `objectKey` starts with `users/{req.user.userId}/{kind}/`. If a caller attempts to confirm another user's key, **403 Forbidden** is returned.
2. **HeadObject Probe**:
   - Must exist in storage. If missing, returns **400 Bad Request** (`object_not_found`).
   - ContentLength must be $\le$ max allowed bytes.
   - ContentType must be in `['image/jpeg', 'image/png', 'image/webp']`.
3. **Key Swap & Cleanup**: Updates `avatarKey` or `bannerKey` on the profile document, and triggers best-effort asynchronous deletion of the old key.

---

## Response

### 200 OK
Media confirmed and profile updated.

```json
{
  "key": "users/usr_9988776655/avatar/3f7a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c.png",
  "url": "https://media.zenuxs.com/users/usr_9988776655/avatar/3f7a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c.png?v=1763429100000"
}
```

---

## Error Responses

### 403 Forbidden (`forbidden`)
Caller attempted to confirm an objectKey belonging to a different user.

```json
{
  "error": {
    "code": "forbidden",
    "message": "Object key does not belong to the authenticated user"
  }
}
```

### 400 Bad Request (`object_not_found`)
Object does not exist in storage (e.g. client never uploaded the file or upload failed).

```json
{
  "error": {
    "code": "object_not_found",
    "message": "Object does not exist in storage"
  }
}
```

---

## Curl Example

```bash
curl -i -X POST http://localhost:4002/me/avatar/confirm \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"objectKey": "users/usr_9988776655/avatar/3f7a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c.png"}'
```
