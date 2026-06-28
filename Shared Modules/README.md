# Shared Modules

Drop-in patterns every blueprint reuses. Copy the JSON snippet into a scenario, or just follow the shape.

---

## The Automation Log Data Store

Create one Make **Data Store** named `Automation Log` and point every blueprint's log modules at it. Recommended structure:

| Field | Type | Notes |
|---|---|---|
| `key` (data structure key) | Text | unique per row; pattern `<scenario>-<id>-<date>` |
| `timestamp` | Text/Date | ISO 8601 |
| `scenario` | Text | folder name of the blueprint |
| `status` | Text | `success` · `error` · `skipped` · `escalated` |
| `record_id` | Text | external id / email |
| `message` | Text | human-readable detail |

[`Utilities/daily-ops-digest`](../Utilities/daily-ops-digest/) reads this store to produce a daily health report.

---

## 1. Retry error handler (`Break`)

Attach to any fragile API module. Retries with backoff, then lets the scenario's `maxErrors` decide.

```json
"onerror": [
  {
    "module": "builtin:Break",
    "version": 1,
    "parameters": {},
    "mapper": { "count": "3", "interval": "5", "retry": true }
  }
]
```

- `count` — attempts. `interval` — minutes between them.
- Use **3 / 5** for read APIs, **3 / 10** for heavier writes (email, CRM writes).

## 2. Log-and-continue handler (`Resume`)

For non-critical steps (notifications) where failure shouldn't kill the run.

```json
"onerror": [
  { "module": "builtin:Resume", "version": 1, "mapper": { "value": "skipped" } }
]
```

Or log first, then resume (used in the AI blueprints when JSON parsing fails):

```json
"onerror": [
  {
    "module": "datastore:AddReplaceRecord",
    "parameters": { "datastore": "1" },
    "mapper": {
      "key": "err-{{1.id}}-{{now}}",
      "record": { "timestamp": "{{now}}", "scenario": "<name>", "status": "error", "record_id": "{{1.id}}", "message": "<what failed>" }
    },
    "onerror": [ { "module": "builtin:Resume", "version": 1, "mapper": { "value": "logged" } } ]
  }
]
```

## 3. Logger module

The standard final step on every path:

```json
{
  "module": "datastore:AddReplaceRecord",
  "parameters": { "datastore": "1" },
  "mapper": {
    "key": "<scenario>-{{record_id}}",
    "overwrite": true,
    "record": {
      "timestamp": "{{now}}",
      "scenario": "<scenario>",
      "status": "success",
      "record_id": "{{...}}",
      "message": "{{...}}"
    }
  }
}
```

## 4. Notifier (Slack, with safe failure)

```json
{
  "module": "slack:CreateMessage",
  "mapper": { "channel": "{{connection.channelId}}", "text": "..." },
  "onerror": [ { "module": "builtin:Resume", "version": 1, "mapper": { "value": "notify-skipped" } } ]
}
```

Notifications are always **routed or terminal** — never inline in the happy path — so alerting can be toggled without touching core logic.

---

## When to use which directive

| Directive | Use when |
|---|---|
| `Break` | Transient failure; retrying will likely succeed (rate limit, 5xx, timeout). |
| `Resume` | The step is optional; failure is acceptable (a notification). |
| `Ignore` | A "not found" is a valid outcome (e.g. dedup search returns nothing). |
| `Rollback` | A multi-write must be atomic; undo partial work. |
| `Commit` | Force-persist work before an expected break. |
