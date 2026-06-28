# The Log Sink

Every blueprint logs to a single HTTP endpoint so you have one auditable trail across the whole library. The blueprints reference it as `https://YOUR_LOG_SINK`. You decide what's behind it — the contract is tiny.

## Contract

### `POST /append`
Each run sends one structured row:
```json
{
  "timestamp": "2026-06-28T14:03:11Z",
  "scenario": "lead-capture-to-crm",
  "status": "success",          // success | error | skipped | escalated
  "record_id": "dana@acme.io",
  "message": "Contact upserted (id 8842)"
}
```
Respond `2xx`. (If it fails, the blueprint's `Resume` handler swallows it — logging never breaks the run.)

### `GET /summary?since=<ISO timestamp>`
Used by [Daily Ops Digest](../Utilities/daily-ops-digest/). Returns counts since the timestamp:
```json
{ "success": 142, "error": 3, "escalated": 1 }
```

## Three ways to implement it

### Option A — Make Data Store (stays inside Make)
1. Create one **Data Store** named `Automation Log` with the fields above.
2. Build a tiny "Logger" scenario: **Webhook** → **Data Store › Add a record**. Use its webhook URL as `YOUR_LOG_SINK/append`.
3. Build a "Summary" scenario: **Webhook** → **Data Store › Search/Count records** (filter by `timestamp >= since`, group by `status`) → **Webhook response**. Use its URL for `/summary`.

> This keeps the *unverified* Data Store module isolated to **two** small scenarios you build once in the UI — instead of baking a possibly-stale module identifier into all 10 blueprints. Every other blueprint just does a portable HTTP POST.

### Option B — Google Sheet
Bind a Google Apps Script Web App: `doPost` appends a row, `doGet` returns the summary. Point `YOUR_LOG_SINK` at the published `/exec` URL. Human-browsable.

### Option C — Hosted logging / Airtable / your own API
Anything that accepts the POST and answers the summary GET: Airtable (via its REST API), BetterStack, a 20-line serverless function, etc.

## Why a sink instead of a Data Store module in every blueprint

The portfolio's blueprints stay **100% portable** (only `http` + `builtin` modules, which import into any account). The one place that touches a vendor-specific module — the Data Store — is a single scenario you wire to your own store. Swap the backend without editing a single blueprint.
