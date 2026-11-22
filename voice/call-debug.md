# GET /calls/:id/debug

Internal diagnostic inspection endpoint that compares persisted ScyllaDB call state against live WebRTC room state in LiveKit to detect state divergence.

---

## Endpoint Details

- **Method**: `GET`
- **Path**: `/calls/:id/debug`
- **Authentication**: `X-Internal-Token` header. Missing or mismatched token returns `401 Unauthorized`.
- **Rate Limit**: None (internal maintenance and observability traffic)

---

## Divergence Classifications

The response computes a `divergence` indicator:
- `"none"`: Persisted database state matches live WebRTC room reality.
- `"stuck_call"`: Persisted state is `'accepted'`, but 0 participants are connected in LiveKit (indicates an abandoned session awaiting reconciliation).
- `"orphan_room"`: Persisted state is `'ended'`, but participants are still connected to the LiveKit room.

---

## Example Request

```bash
curl http://localhost:4004/calls/1f9543f3-1217-4462-8bd5-260b9bed1156/debug \
  -H "X-Internal-Token: dev-internal-token-change-me"
```

## Example Response

```json
{
  "call": {
    "id": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "callerId": "11111111-1111-4111-8111-111111111111",
    "calleeId": "22222222-2222-4222-8222-222222222222",
    "state": "accepted",
    "roomName": "1f9543f3-1217-4462-8bd5-260b9bed1156",
    "createdAt": "2026-11-22T16:30:00.000Z",
    "acceptedAt": "2026-11-22T16:30:05.000Z",
    "endedAt": null,
    "endedBy": null,
    "endReason": null
  },
  "livekit": {
    "roomExists": true,
    "participantCount": 2,
    "participants": [
      {
        "identity": "11111111-1111-4111-8111-111111111111",
        "sid": "PA_xxxx",
        "joinedAt": 1795378005
      },
      {
        "identity": "22222222-2222-4222-8222-222222222222",
        "sid": "PA_yyyy",
        "joinedAt": 1795378006
      }
    ]
  },
  "divergence": "none"
}
```
