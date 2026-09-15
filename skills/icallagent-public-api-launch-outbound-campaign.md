---
name: Launch an outbound calling campaign
description: Create an iCallAgent outbound campaign bound to a Live voice agent, then queue contacts into it to place calls.
api: iCallAgent Public API
base_url: https://api.icallagent.com/api/public/v1/
auth: Bearer token (personal API key ic_live_… or OAuth2 access token)
operations:
  - listAgents
  - listPhoneNumbers
  - createCampaign
  - createContact
---

# Launch an outbound calling campaign

Place outbound AI-voice calls by creating a campaign bound to a Live agent, then queuing
contacts into it. All requests send `Authorization: Bearer <token>`.

## Steps

1. **Pick the agent** — `GET /agents/` (`listAgents`). Returns `{ "results": [ { "id", "name" } ] }`
   for Live agents only. Note the agent `id`; it becomes the campaign's `widget_id`. Agents are
   created/edited in the dashboard, not the API.

2. **Pick an outbound number (optional)** — `GET /phone-numbers/` (`listPhoneNumbers`). Use a
   number `id` as the campaign's `phone_number_id` to set caller ID; otherwise pass `caller_id`
   directly. A `phone_number_id` outside your workspace returns **404**.

3. **Create the campaign** — `POST /campaigns/` (`createCampaign`). Required: `name` and
   `widget_id`. Useful optional fields: `phone_number_id`/`caller_id`, `max_concurrent_calls`
   (silently clamped to your entitlement — read the applied value back), `retry_attempts` /
   `retry_delay_minutes`, and a dial window (`timezone`, `start_date`/`end_date`,
   `start_time`/`end_time`, `weekdays`, `excluded_dates`). Returns **201**.

4. **Queue contacts** — `POST /contacts/` (`createContact`). Required: `phone`. Attach to the
   campaign with `campaign_id` (preferred) or `campaign` name (exact match; ambiguous names
   return **409**). Set `external_id` as a per-campaign idempotency key — replaying the same
   `external_id` returns the original call with **200** instead of creating a duplicate (a new
   contact returns **201**).

## Conventions & error handling

- **Errors** are a flat body `{"detail": "..."}` (not RFC 9457); read the HTTP status:
  **400** missing/invalid field, **401** bad token, **404** unknown/cross-workspace id,
  **409** ambiguous campaign name or campaign not accepting contacts, **422** compliance gate
  blocked the number (blacklist / do-not-call / revoked consent).
- **Idempotency**: only `createContact` supports replay protection, via `external_id`. Reuse a
  stable `external_id` per contact-per-campaign so retries don't double-dial.
- **Pagination** on list calls: `limit` (default 100, max 200) and `offset` (default 0); rows
  are under `results`.
