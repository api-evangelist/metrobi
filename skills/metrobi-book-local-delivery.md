---
name: Book a local courier delivery with Metrobi
description: Estimate the price, create a same-day local delivery, and receive live status updates via webhook using the Metrobi Delivery API.
api: openapi/metrobi-delivery-api-openapi.json
operations: [delivery-estimation, create-a-delivery, get-a-delivery-status]
---

# Book a local courier delivery with Metrobi

Use this skill to arrange a local same-day courier delivery (one pickup, one or more dropoffs) through the Metrobi Delivery API.

## Auth
- Base URL: `https://delivery-api.metrobi.com/api/`
- Every request sends the header `x-api-key: <YOUR_API_KEY>` (generate the key in the Metrobi sender dashboard after creating an account at https://metrobi.com/register/).

## Steps

1. **(Optional) Estimate first** — call `delivery-estimation` (`POST /v1/delivery_estimate`) with `cargo_size`, `pickup_time` (`date` YYYY-MM-DD + `time` HH:mm), `pickup_stop`, and `dropoff_stop` to get a price and distance estimate before committing.

2. **Create the delivery** — call `create-a-delivery` (`POST /v1/delivery`). Required fields: `cargo_size` (`extra_small` | `medium` | `large` | `cargo_van`), `pickup_time`, `pickup_stop` (`name` + `address` required), and `dropoff_stop` (`name` + `address` required). Include the receiver's `dropoff_stop.contact.phone`/`email` so Metrobi can send delivery notifications. To get live callbacks, set `webhook_url` to a public HTTPS endpoint that accepts POST and returns 2xx.
   - The response returns `response.data.delivery_id` — persist it.

3. **Track status** — either handle the webhook callbacks (fired on dropoff in-progress / completed / cancelled, carrying the full delivery JSON incl. proof-of-delivery photos), or poll `get-a-delivery-status` (`GET /v1/delivery/{delivery_id}`) with the saved `delivery_id`.

## Rules
- `cargo_size` drives driver/vehicle eligibility; `extra_small` allows only one stop, other sizes allow up to 50 stops.
- Timestamps in responses are epoch milliseconds.
- Errors return HTTP 400 with `{"success": false, "response": {"message": "..."}}` (not RFC 9457). See `errors/metrobi-problem-types.yml`.
- No idempotency key is supported — do not blind-retry `create-a-delivery`; on a network failure, reconcile with `list-deliveries` before retrying.
