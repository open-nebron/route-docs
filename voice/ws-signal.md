# Real-Time Voice Signaling Protocol (`ws:signal:{userId}`)

Documentation for real-time WebRTC push signaling in Nebron via Redis Pub/Sub channels and `chat-service` WebSocket forwarder.

---

## Architectural Overview

`voice-service` publishes signaling events to Redis channels using the format `ws:signal:{userId}`.
`chat-service` maintains persistent WebSocket connections with active client devices. When a message is published to `ws:signal:{userId}`, `chat-service` forwards the JSON frame verbatim to all connected sockets of that user.

```
┌────────────────┐      Redis PUBLISH       ┌────────────────┐    WebSocket Push   ┌────────────────┐
│ voice-service  │ ───────────────────────> │  chat-service  │ ──────────────────> │ Client Device  │
│                │   ws:signal:{userId}     │ (forwarder)    │                     │  (Browser/App) │
└────────────────┘                          └────────────────┘                     └────────────────┘
```

---

## Signaling Message Frame Formats

All frames delivered on `ws:signal:{userId}` adhere to:
```json
{
  "type": "<string>",
  "payload": { ... }
}
```

### 1. `call.ringing` (Pushed to Callee)
Sent immediately when a caller initiates a call.
```json
{
  "type": "call.ringing",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "callerId": "11111111-1111-4111-8111-111111111111",
    "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "startedAt": "2026-11-22T16:30:00.000Z"
  }
}
```

### 2. `call.dialing` (Pushed to Caller)
Sent immediately to confirm outbound ringing state.
```json
{
  "type": "call.dialing",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "calleeId": "22222222-2222-4222-8222-222222222222"
  }
}
```

### 3. `call.accepted` (Pushed to Caller)
Sent when the callee answers the call. Contains LiveKit credentials for the caller.
```json
{
  "type": "call.accepted",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "callerToken": "eyJhbGciOi...",
    "livekitUrl": "ws://localhost:7880",
    "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156"
  }
}
```

### 4. `call.connected` (Pushed to Callee)
Sent to the callee upon accepting the call. Contains LiveKit credentials for the callee.
```json
{
  "type": "call.connected",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "calleeToken": "eyJhbGciOi...",
    "livekitUrl": "ws://localhost:7880",
    "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156"
  }
}
```

### 5. `call.rejected` (Pushed to Caller)
Sent when the callee rejects or declines the call.
```json
{
  "type": "call.rejected",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "calleeId": "22222222-2222-4222-8222-222222222222",
    "reason": "busy"
  }
}
```

### 6. `call.cancelled` (Pushed to Callee)
Sent when the caller hangs up before the callee accepts.
```json
{
  "type": "call.cancelled",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "callerId": "11111111-1111-4111-8111-111111111111",
    "reason": "cancelled"
  }
}
```

### 7. `call.ended` (Pushed to Active Peer)
Sent when either party terminates an accepted call.
```json
{
  "type": "call.ended",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "endedBy": "11111111-1111-4111-8111-111111111111",
    "reason": "hangup"
  }
}
```

### 8. `call.missed` (Pushed to Both Parties)
Sent when a ringing call exceeds the timeout threshold without being answered.
```json
{
  "type": "call.missed",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156"
  }
}
```

### 9. `call.peer_disconnected` (Pushed to Remaining Participant)
Sent when one participant unexpectedly disconnects from the WebRTC room in LiveKit while the other participant is still connected.
```json
{
  "type": "call.peer_disconnected",
  "payload": {
    "callId": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "userId": "11111111-1111-4111-8111-111111111111"
  }
}
```
