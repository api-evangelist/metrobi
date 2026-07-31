---
name: List, read, and cancel Metrobi deliveries
description: Reconcile and manage existing deliveries with the Metrobi Delivery API — list by date range, read one, and cancel with a reason.
api: openapi/metrobi-delivery-api-openapi.json
operations: [list-deliveries, get-a-delivery-status, cancel-a-delivery]
---

# List, read, and cancel Metrobi deliveries

Use this skill to manage deliveries you have already created.

## Auth
- Base URL: `https://delivery-api.metrobi.com/api/`
- Header on every request: `x-api-key: <YOUR_API_KEY>`.

## Steps

1. **List deliveries** — call `list-deliveries` (`GET /v1/delivery`) with query params `fromDate` and `toDate` (both `YYYY-MM-DD`). Results are filtered by pickup-time within the inclusive range; there is no cursor/offset pagination, so widen or narrow the date window to scope results.

2. **Read one** — call `get-a-delivery-status` (`GET /v1/delivery/{delivery_id}`) with a `delivery_id` from the list to fetch the current status and stops.

3. **Cancel** — call `cancel-a-delivery` (`DELETE /v1/delivery/{delivery_id}`) with a JSON body `{"reason": "..."}` (required). The response echoes the delivery with each stop `status` set to `CANCELLED` and any `cancellation.fee`.

## Rules
- Stop `status` values: `SUBMITTED`, `TODO`, `INPROGRESS`, `COMPLETED`, `CANCELLED`; stop `type` values: `PICKUP`, `DROPOFF`, `RETURN`.
- Cancelling may incur a `cancellation.fee` depending on delivery progress — surface it to the user.
- A missing/invalid id returns HTTP 400 `{"success": false, "response": {"message": "Delivery not found"}}`. See `errors/metrobi-problem-types.yml`.
- Timestamps are epoch milliseconds.
