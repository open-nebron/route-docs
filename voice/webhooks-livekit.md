# POST /webhooks/livekit

Receives and processes WebRTC room lifecycle events and participant disconnect events pushed directly by LiveKit server.

---

## Endpoint Details

- **Method**: `POST`
- **Path**: `/webhooks/livekit`
- **Authentication**: LiveKit JWT signature verification (via `Authorization` header). Does not use user Bearer tokens.
- **Content-Types**: `application/webhook+json`, `application/json`
- **Rate Limit**: None (server-to-server internal traffic)

---

## Authentication & Verification

LiveKit signs webhooks using `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`. The `Authorization` header contains an HS256 JWT whose claims include a `sha256` digest of the raw request body.

The service verifies this using `WebhookReceiver.receive(rawBody, authHeader)`:
- If `Authorization` header is missing or signature verification fails -> returns `401 Unauthorized` with `{ error: { code: "invalid_webhook_signature", message: "..." } }`.
- Always returns `200 OK` on valid signatures (even if events are unhandled or stale) to prevent LiveKit from retrying delivery indefinitely.

---

## Handled Events

### 1. `room_finished`
Triggered by LiveKit when a room is destroyed or closed (e.g. empty room timeout).
- **Behavior**:
  - Finds call by `id = event.room.name`.
  - If state is `ringing` or `accepted`:
    - Transitions call to `state='ended'`, `endReason='livekit_closed'`, `endedBy=null`, `endedAt=now`.
    - Publishes `call.ended` signal on `ws:signal:{callerId}` and `ws:signal:{calleeId}` with `reason: "livekit_closed"`.
    - Publishes `voice.call.ended` event to Redis Stream `nebron.events`.
  - If state is already terminal (`ended`, `rejected`, `cancelled`, `missed`, `error`): ignores (idempotent).

### 2. `participant_left`
Fast path for detecting client disconnects.
- **Behavior**:
  - Finds call by `id = event.room.name`.
  - If call is not in `accepted`: ignores.
  - Queries LiveKit for current room participants (`roomService.listParticipants`):
    - If `count === 0` (both left): immediately force-ends the call (`livekit_closed`), publishes WS signals, and emits `voice.call.ended` event.
    - If `count === 1`: publishes `call.peer_disconnected` on `ws:signal:{remainingUserId}` to alert the surviving client.

### 3. `room_started`
- Logs room creation diagnostic information. No state change.

### 4. `participant_joined`
- Logs participant join diagnostic information. No state change.

---

## Example Request (from LiveKit Server)

```http
POST /webhooks/livekit HTTP/1.1
Host: localhost:4004
Authorization: eyJhbGciOiJIUzI1NiJ9...
Content-Type: application/webhook+json

{
  "event": "room_finished",
  "room": {
    "sid": "RM_xxxx",
    "name": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "emptyTimeout": 300,
    "maxParticipants": 2,
    "creationTime": 1795378000,
    "numParticipants": 0
  }
}
```

## Example Response

```json
{
  "ok": true
}
```
