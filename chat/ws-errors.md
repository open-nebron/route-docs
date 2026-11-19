# WebSocket Close Codes and Error Reference

This document catalogs the WebSocket close codes and in-band error frames utilized by the Nebron Chat Service (`chat-service`).

---

## 1. Custom WebSocket Close Codes (4400–4499)

In accordance with RFC 6455 §7.4.2, the 4000–4999 range is reserved for application-specific close codes. The Nebron Chat Service defines the following custom closure codes:

| Code | Reason String | Trigger Condition | Recommended Client Action |
|---|---|---|---|
| **`4401`** | `invalid_token` | Missing token in query string, malformed JWT, expired token, signature verification failure against JWKS, issuer mismatch (`!= nebron-auth`), or audience does not include `chat`. | Refresh access token using `POST /auth/refresh` at `auth-service` and reconnect with new token. |
| **`4403`** | `session_revoked` | The user's active refresh session has been revoked (e.g. user logged out, session terminated by security policy, token reuse detected, or session revocation event `auth.session.revoked` received). | Prompt user to log in again. Do not automatically retry with the same session credentials. |
| **`4408`** | `too_many_connections` | The user already has **5** active concurrent WebSocket connections. When a 6th socket connects, the server terminates the oldest socket with this code. | If user has multiple browser tabs/devices open, warn the user. Avoid reconnect loops from dormant background tabs. |
| **`4409`** | `frame_too_large` | An individual WebSocket frame sent by the client exceeded **32 KB** (32,768 bytes). | Truncate or paginate payload on client side. Attachments must use presigned media upload URLs, never raw WS frames. |

---

## 2. Standard WebSocket Close Codes

| Code | Standard Name | Usage in Chat Service |
|---|---|---|
| **`1000`** | `Normal Closure` | Clean client-initiated disconnect or graceful server shutdown (`SIGTERM`/`SIGINT`). |
| **`1006`** | `Abnormal Closure` | Socket closed abnormally without sending a close frame (e.g. network drop, process killed). |
| **`1011`** | `Internal Error` | Unhandled server exception during handshake or upgrade. |

---

## 3. In-Band Error Frames

When a connected socket sends a frame that fails schema validation or business rules (and does not necessitate closing the connection), the server replies with an in-band error frame containing an `ackId` matching the request's `id`:

```json
{
  "type": "error",
  "payload": {
    "code": "<code>",
    "message": "<human_readable_message>"
  },
  "ackId": "msg-send-01"
}
```

### In-Band Error Codes:

| Code | HTTP Equivalent | Description |
|---|---|---|
| `rate_limited` | 429 Too Many Requests | Rate limit exceeded on this socket (`message.send`: 10/s, `typing.*`: 10/s, `message.read`: 20/s). |
| `invalid_payload` | 400 Bad Request | Frame is malformed JSON, missing required fields, or contains null bytes. |
| `not_a_member` | 403 Forbidden | Authenticated user is not a participant in the target conversation. |
| `empty_message` | 400 Bad Request | Message body contains 0 characters after whitespace trimming. |
| `message_too_long` | 400 Bad Request | Message body exceeds 4000 characters. |
| `message_not_in_conversation` | 404 Not Found | Referenced messageId does not exist in the specified conversation. |
| `internal_error` | 500 Internal Server Error | Unexpected server error during frame processing. |
