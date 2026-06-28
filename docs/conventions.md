# Engineering conventions

Every blueprint in this library is built against the same standards. If you contribute, your scenario must meet all five. This is what separates a portfolio of demos from a portfolio of *production* automations.

---

## 1. Module ID & layout discipline

- Module IDs are **sequential integers** starting at `1`, in execution order.
- The trigger is always module `1`.
- Designer coordinates (`metadata.designer.x/y`) flow left→right in ~300px steps so an imported scenario lays out cleanly instead of stacking on top of itself.
- Routers (`builtin:BasicRouter`) branch top→bottom; the "happy path" is the first route.

## 2. Retries (self-healing on transient failure)

Any module that talks to an external API over the network is **fragile** — it can fail for reasons that have nothing to do with your logic (rate limits, 5xx, timeouts). Those modules get an **error handler** with a `Break` directive:

```jsonc
"onerror": [
  {
    "module": "builtin:Break",
    "mapper": {
      "count": "3",        // up to 3 attempts
      "interval": "5"       // 5 minutes between attempts
    }
  }
]
```

- Use `Break` for *transient* failures (retry makes sense).
- Use `Resume` / `Ignore` only when a missing record is an acceptable outcome.
- Use `Rollback` when a multi-step write must be atomic.
- **Never** leave a raw HTTP module with no error handler — an unhandled error halts the whole scenario and (on instant triggers) can drop the bundle.

## 3. Logging (an audit trail you can query)

Every blueprint writes a **structured log row** at the end of each run — success *and* failure. Standard schema:

| Field | Example |
|---|---|
| `timestamp` | `2026-06-28T14:03:11Z` |
| `scenario` | `lead-capture-to-crm` |
| `status` | `success` \| `error` \| `skipped` |
| `record_id` | external id or email |
| `message` | human-readable detail |

Sink options (pick one per environment): **Make Data Store** (fastest, no external app) or **Google Sheets** (human-browsable). The blueprints default to Data Store; swap the final module if you prefer Sheets. See [Shared Modules](../Shared%20Modules/).

## 4. Notifications (humans stay in the loop)

- **Failures** after retries are exhausted → notify a Slack channel (or email fallback) with scenario name, record id, and the error message.
- **High-value successes** (a deal created, a contract signed) → optional success ping.
- Notifications are routed, never inline in the happy path, so they can be toggled without touching core logic.

## 5. Secrets & sanitization

- Blueprints are committed **without** connection credentials. Make strips most secrets on export; we additionally null any `__IMTCONN__` account hints.
- No API keys, tokens, webhook URLs, or real customer data in committed JSON.
- Per-blueprint READMEs list the connections you must wire after import, and the env/config values to set.

---

## Naming

- Folder: `kebab-case` matching the scenario's job (`stale-deal-reminder`).
- Scenario `name` inside the JSON: Title Case, human-readable (`Stale Deal Reminder`).
- Each blueprint folder contains exactly: `blueprint.json`, `README.md`, and (optional) `diagram.md`.

## The standard skeleton

```
Trigger → Validate/Route → Core action(+error handler) → Log → Notify
```

Read [architecture.md](architecture.md) for how each category specializes this skeleton.
