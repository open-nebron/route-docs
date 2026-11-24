# Users API Documentation (`/users`)

This document provides endpoint definitions, schemas, and curl examples for user discovery and public profile lookups in `core-backend`.

---

## 1. Search Users

Search registered user profiles by prefix on `username` or display `name`.

- **Method**: `GET`
- **Path**: `/users/search?q=<query>&limit=<limit>`
- **Authentication**: Bearer Token (Required)
- **Rate Limit**: 30 requests / minute per user

### Request Query Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `q` | string | Yes | 1-50 chars prefix to search (case-insensitive) |
| `limit` | integer | No | Max users returned (default: 20, max: 50) |

### Response (`200 OK`)

```json
{
  "users": [
    {
      "userId": "67406a091535780a427b5f12",
      "username": "alex",
      "name": "Alex Mercer",
      "avatarUrl": "http://localhost:9000/nebron-media/users/67406a091535780a427b5f12/avatar/uuid.jpg"
    }
  ]
}
```

### Curl Example

```bash
curl -X GET "http://localhost:4002/users/search?q=al&limit=10" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

---

## 2. Get User By Username

Fetch a user's public profile summary by their unique `@username`.

- **Method**: `GET`
- **Path**: `/users/by-username/:username`
- **Authentication**: Bearer Token (Required)
- **Rate Limit**: 60 requests / minute per user

### Response (`200 OK`)

```json
{
  "userId": "67406a091535780a427b5f12",
  "username": "alex",
  "name": "Alex Mercer",
  "avatarUrl": "http://localhost:9000/nebron-media/users/67406a091535780a427b5f12/avatar/uuid.jpg",
  "bio": "Building real-time communications at Nebron"
}
```

### Curl Example

```bash
curl -X GET "http://localhost:4002/users/by-username/alex" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

---

## 3. Get Public Profile by User ID

Fetch a user's public profile fields by `userId`. Used by avatar popovers and message header cards.

- **Method**: `GET`
- **Path**: `/users/:userId`
- **Authentication**: Bearer Token (Required)
- **Rate Limit**: 60 requests / minute per user

### Response (`200 OK`)

```json
{
  "userId": "67406a091535780a427b5f12",
  "username": "alex",
  "name": "Alex Mercer",
  "avatarUrl": "http://localhost:9000/nebron-media/users/67406a091535780a427b5f12/avatar/uuid.jpg",
  "bannerUrl": "http://localhost:9000/nebron-media/users/67406a091535780a427b5f12/banner/uuid.jpg",
  "bio": "Building real-time communications at Nebron",
  "about": "Full-stack engineer passionate about low-latency systems.",
  "createdAt": "2026-11-20T10:00:00.000Z"
}
```

### Curl Example

```bash
curl -X GET "http://localhost:4002/users/67406a091535780a427b5f12" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```
