# Platform Metadata (`/meta`)

Public endpoint returning configuration metadata, supported languages, theme options, accepted file formats, and system limits. Clients use this to render settings, onboarding, and upload components.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/meta`
- **Authentication**: None (Public)
- **Cache-Control**: `public, max-age=3600`
- **Service**: `core-api` (Port: `4002`)

---

## Request

### Headers
None required.

---

## Response

### 200 OK
Metadata returned with caching headers.

```json
{
  "languages": [
    { "code": "en", "name": "English" },
    { "code": "hi", "name": "Hindi" },
    { "code": "ta", "name": "Tamil" },
    { "code": "te", "name": "Telugu" },
    { "code": "ml", "name": "Malayalam" },
    { "code": "kn", "name": "Kannada" },
    { "code": "bn", "name": "Bengali" },
    { "code": "mr", "name": "Marathi" },
    { "code": "gu", "name": "Gujarati" },
    { "code": "pa", "name": "Punjabi" },
    { "code": "ur", "name": "Urdu" }
  ],
  "themes": ["light", "dark", "system"],
  "defaultLanguage": "en",
  "limits": {
    "avatarMaxBytes": 5242880,
    "bannerMaxBytes": 10485760,
    "bioMaxChars": 300,
    "aboutMaxChars": 2000,
    "nameMaxChars": 80
  },
  "acceptedImageTypes": ["image/jpeg", "image/png", "image/webp"]
}
```

---

## Curl Example

```bash
curl -i http://localhost:4002/meta
```
