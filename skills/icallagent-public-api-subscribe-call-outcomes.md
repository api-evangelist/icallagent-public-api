---
name: Subscribe to call outcomes
description: Register a webhook to receive iCallAgent call.completed events and process final call outcomes reliably.
api: iCallAgent Public API
base_url: https://api.icallagent.com/api/public/v1/
auth: Bearer token (personal API key ic_live_… or OAuth2 access token)
operations:
  - subscribeWebhook
  - unsubscribeWebhook
---

# Subscribe to call outcomes

Receive a POST whenever a call finishes, so you can log outcomes or trigger follow-up.

## Steps

1. **Subscribe** — `POST /webhooks/subscribe` (`subscribeWebhook`). Body: `{ "url": "<your
   https callback>" }`. Returns **201**. The `url` is where every `call.completed` event is
   delivered.

2. **Handle `call.completed`** — iCallAgent POSTs this to your URL when a call reaches a
   terminal state. Key fields: `call_id`, `status` (only `completed` or `failed`), `direction`,
   `agent` (`{id,name}` or null), `contact` (`{id,name,phone}`, never null — may have empty
   `phone`), `campaign` (`{id,name}` or null for ad hoc/browser/inbound), `answered_at`,
   `ended_at`, `duration_seconds`, `hangup_cause`. Respond fast with a **2xx**.

3. **Dedupe** — delivery is **at-least-once**. Dedupe on `call_id` if you need strict
   idempotency. A call about to retry does NOT fire the event; only its final outcome does.

4. **Unsubscribe** — `DELETE /webhooks/{id}/` (`unsubscribeWebhook`) to stop delivery.
   Returns **204**.

## Notes

- Check for an empty `contact.phone` rather than a missing `contact` object (browser / ad hoc
  calls fall back to `{name: "<call target label>", phone: ""}`).
- Errors follow the flat `{"detail": "..."}` shape; **400** on a missing/invalid `url`.
