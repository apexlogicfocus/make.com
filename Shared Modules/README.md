# Shared Modules

Drop-in patterns every blueprint reuses. The library is **HTTP-first** — every external call is an `http:ActionSendData` module, so these patterns work in any Make account without depending on a specific app's module identifier. Copy a snippet, or just follow the shape.

> Background on why HTTP-first: [docs/conventions.md → §0](../docs/conventions.md#0-integration-agnostic-by-design-http-first).

---

## The Automation Log sink

Every blueprint POSTs its log rows to `https://YOUR_LOG_SINK/append`, and the digest reads `GET /summary`. You implement the sink once (Make Data Store, Google Sheet, Airtable, …). Schema + three implementation options: [docs/log-sink.md](../docs/log-sink.md).

| Field | Type | Notes |
|---|---|---|
| `timestamp` | ISO 8601 | `{{now}}` |
| `scenario` | Text | folder name of the blueprint |
| `status` | Text | `success` · `error` · `skipped` · `escalated` |
| `record_id` | Text | external id / email |
| `message` | Text | human-readable detail |

---

## 1. Retry error handler (`Break`)

Attach to any fragile HTTP module. Retries with backoff, then lets the scenario's `maxErrors` decide. Requires `handleErrors: true` on the HTTP module so non-2xx responses throw into the handler.

```json
"onerror": [
  { "module": "builtin:Break", "version": 1, "parameters": {}, "mapper": { "count": "3", "interval": "5", "retry": true } }
]
```

- `count` — attempts. `interval` — minutes between them.
- Use **3 / 5** for reads, **3 / 10** for heavier writes (email, CRM writes).

## 2. Log-and-continue handler (`Resume`)

For non-critical steps (notifications, logging) where failure shouldn't kill the run.

```json
"onerror": [
  { "module": "builtin:Resume", "version": 1, "mapper": { "value": "skipped" } }
]
```

Or log first, then resume (used in the AI blueprints when a downstream call fails):

```json
"onerror": [
  {
    "module": "http:ActionSendData",
    "version": 3,
    "parameters": { "handleErrors": true, "useNewZLibDeCompress": true },
    "mapper": {
      "url": "https://YOUR_LOG_SINK/append",
      "method": "post",
      "headers": [ { "name": "Content-Type", "value": "application/json" } ],
      "bodyType": "raw", "parseResponse": false, "contentType": "application/json",
      "data": "{ \"timestamp\": \"{{now}}\", \"scenario\": \"<name>\", \"status\": \"error\", \"record_id\": \"{{...}}\", \"message\": \"<what failed>\" }"
    },
    "onerror": [ { "module": "builtin:Resume", "version": 1, "mapper": { "value": "logged" } } ]
  }
]
```

## 3. Logger (HTTP POST)

The standard final step on every path:

```json
{
  "module": "http:ActionSendData",
  "version": 3,
  "parameters": { "handleErrors": true, "useNewZLibDeCompress": true },
  "mapper": {
    "url": "https://YOUR_LOG_SINK/append",
    "method": "post",
    "headers": [ { "name": "Content-Type", "value": "application/json" } ],
    "bodyType": "raw", "parseResponse": false, "contentType": "application/json",
    "data": "{ \"timestamp\": \"{{now}}\", \"scenario\": \"<name>\", \"status\": \"success\", \"record_id\": \"{{...}}\", \"message\": \"{{...}}\" }"
  },
  "onerror": [ { "module": "builtin:Resume", "version": 1, "mapper": { "value": "logged" } } ]
}
```

## 4. Notifier (Slack incoming webhook over HTTP)

```json
{
  "module": "http:ActionSendData",
  "version": 3,
  "parameters": { "handleErrors": true, "useNewZLibDeCompress": true },
  "mapper": {
    "url": "https://hooks.slack.com/services/REPLACE_WITH_YOUR_SLACK_WEBHOOK",
    "method": "post",
    "headers": [ { "name": "Content-Type", "value": "application/json" } ],
    "bodyType": "raw", "parseResponse": false, "contentType": "application/json",
    "data": "{ \"text\": \"...\" }"
  },
  "onerror": [ { "module": "builtin:Resume", "version": 1, "mapper": { "value": "notify-skipped" } } ]
}
```
Create a Slack incoming webhook at *api.slack.com → Your Apps → Incoming Webhooks*. Swap the URL for a Discord/Teams webhook and it still works.

## 5. LLM call (HTTP) + built-in JSON parsing

Call the provider's chat-completions endpoint, force JSON, then read fields with `parseJSON()` — no parse module required.

```json
{
  "module": "http:ActionSendData",
  "version": 3,
  "parameters": { "handleErrors": true, "useNewZLibDeCompress": true },
  "mapper": {
    "url": "https://api.openai.com/v1/chat/completions",
    "method": "post",
    "headers": [
      { "name": "Authorization", "value": "Bearer YOUR_OPENAI_API_KEY" },
      { "name": "Content-Type", "value": "application/json" }
    ],
    "bodyType": "raw", "parseResponse": true, "contentType": "application/json",
    "data": "{ \"model\": \"gpt-4o-mini\", \"temperature\": 0, \"response_format\": { \"type\": \"json_object\" }, \"messages\": [ … ] }"
  },
  "onerror": [ { "module": "builtin:Break", "version": 1, "mapper": { "count": "3", "interval": "3", "retry": true } } ]
}
```
Downstream, read fields with `{{parseJSON(2.data.choices[1].message.content).fieldName}}`. (Make arrays are 1-indexed, so `choices[1]`.) Point the URL at Anthropic or your own gateway to switch providers.

---

## When to use which directive

| Directive | Use when |
|---|---|
| `Break` | Transient failure; retrying will likely succeed (rate limit, 5xx, timeout). |
| `Resume` | The step is optional; failure is acceptable (a notification, a log write). |
| `Ignore` | A "not found" is a valid outcome (e.g. dedup search returns nothing). |
| `Rollback` | A multi-write must be atomic; undo partial work. |
| `Commit` | Force-persist work before an expected break. |

## On the "HTTP (legacy)" label

`http:ActionSendData` may show as **"HTTP (legacy)"** in newer Make UIs. It imports and runs correctly everywhere — the label is cosmetic. It's used because it is the most universally compatible HTTP module across all Make accounts.
