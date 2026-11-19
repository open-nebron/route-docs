# Request Media Upload URL (`/me/avatar/upload-url` & `/me/banner/upload-url`)

Requests a direct-to-storage presigned POST upload slot for an avatar ($\le 5$ MB) or banner ($\le 10$ MB) image. The client then uploads the binary multipart/form-data payload directly to Cloudflare R2 / S3 without proxying bytes through `core-api`.

---

## Route Overview

- **Methods**: `POST`
- **Paths**:
  - `/me/avatar/upload-url` (Avatar: max 5 MB)
  - `/me/banner/upload-url` (Banner: max 10 MB)
- **Authentication**: `Bearer <accessToken>`
- **Rate Limit**: 5 requests per minute per user
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
| Header | Value | Required | Description |
|---|---|---|---|
| `Authorization` | `Bearer <ACCESS_TOKEN>` | Yes | Valid RS256 JWT access token |
| `Content-Type` | `application/json` | Yes | Request payload format |

### Body Parameters
| Field | Type | Constraints | Description |
|---|---|---|---|
| `contentType` | string | `image/jpeg` \| `image/png` \| `image/webp` | MIME content type of the image |
| `contentLength` | number | $\le 5242880$ (Avatar) / $\le 10485760$ (Banner) | Expected size in bytes |

Example Body:
```json
{
  "contentType": "image/png",
  "contentLength": 102400
}
```

---

## Response

### 200 OK
Returns a presigned POST slot valid for 5 minutes with strict policy conditions (`content-length-range`, exact `Content-Type`, exact `key`, `success_action_status: 204`).

```json
{
  "uploadUrl": "https://<account>.r2.cloudflarestorage.com/nebron-media",
  "fields": {
    "Content-Type": "image/png",
    "key": "users/usr_9988776655/avatar/3f7a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c.png",
    "success_action_status": "204",
    "bucket": "nebron-media",
    "X-Amz-Algorithm": "AWS4-HMAC-SHA256",
    "X-Amz-Credential": "...",
    "X-Amz-Date": "20261117T000000Z",
    "Policy": "...",
    "X-Amz-Signature": "..."
  },
  "objectKey": "users/usr_9988776655/avatar/3f7a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c.png",
  "expiresAt": "2026-11-17T00:05:00.000Z",
  "maxBytes": 5242880
}
```

---

## Error Responses

### 400 Bad Request (`validation_error`)
Content-type not in whitelist, or file size exceeds maximum limit.

```json
{
  "error": {
    "code": "validation_error",
    "message": "contentType: Content type must be one of image/jpeg, image/png, or image/webp"
  }
}
```

```json
{
  "error": {
    "code": "bad_request",
    "message": "Content length exceeds maximum allowed avatar size of 5242880 bytes"
  }
}
```

### 429 Too Many Requests (`rate_limit_exceeded`)
Exceeded 5 upload-url requests per minute per user.

---

## Upload Execution (Step 2)

Once the upload slot is obtained, upload the file directly to `uploadUrl` via multipart/form-data:

```bash
curl -i -X POST "$UPLOAD_URL" \
  -F "Content-Type=image/png" \
  -F "key=$OBJECT_KEY" \
  -F "success_action_status=204" \
  -F "file=@avatar.png"
```
On success, R2 responds with **204 No Content**. Next, call `/me/avatar/confirm`.
