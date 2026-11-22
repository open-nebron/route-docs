# Call Reconciliation Background Job (`CallReconcileJob`)

Active background reconciliation job in `voice-service` that detects and cleans up stuck calls when clients disconnect ungracefully without webhooks or explicit hangup calls.

---

## The Problem Solved

If both clients drop abruptly (e.g. mobile app force-quit, power outage, network partition), and LiveKit webhooks fail to deliver, the call would remain indefinitely in `state='accepted'` in ScyllaDB. This would cause both users to be permanently locked in `already_in_call` (HTTP 409).

The reconciliation job operates as a periodic self-healing backstop.

---

## Operating Parameters

- **Interval**: `RECONCILE_POLL_INTERVAL_MS` (default: 60,000ms / 60 seconds)
- **Minimum Call Age**: `RECONCILE_MIN_CALL_AGE_SECONDS` (default: 300 seconds / 5 minutes)
- **Batch Size**: `RECONCILE_BATCH_SIZE` (default: 100 calls per cycle)
- **Rate Limit Throttling**: 100ms pause between individual LiveKit participant queries to avoid overloading the SFU control plane.

---

## Reconciliation Logic

1. Query ScyllaDB for all calls with `state = 'accepted'` whose `accepted_at` (or `created_at`) is older than `RECONCILE_MIN_CALL_AGE_SECONDS`.
2. For each candidate call:
   - Query LiveKit Room Service: `roomService.listParticipants(call.roomName)`.
   - If the room does not exist (HTTP 404 / "room not found") or `participants.length === 0`:
     - Perform atomic ScyllaDB update using Lightweight Transactions (LWT):
       ```cql
       UPDATE calls SET state = 'ended', ended_at = ?, ended_by = null, end_reason = 'livekit_closed'
       WHERE id = ? IF state = 'accepted';
       ```
     - If the atomic transaction was applied:
       - Log: `reconciliation: auto-ended stuck call { callId, ageSeconds }`.
       - Publish `ws:signal:{callerId}`: `{ type: "call.ended", payload: { callId, reason: "livekit_closed" } }`.
       - Publish `ws:signal:{calleeId}`: `{ type: "call.ended", payload: { callId, reason: "livekit_closed" } }`.
       - Publish `voice.call.ended` event to Redis Stream `nebron.events`.
   - If LiveKit returns a transient network or server error:
     - Log warning and skip the call; it will be retried on the next reconciliation cycle without modifying call state.
   - If `participants.length > 0`:
     - The call is actively ongoing; no action taken.
