# Engineering conventions

Every blueprint in this library is built against the same standards. If you contribute, your scenario must meet all five. This is what separates a portfolio of demos from a portfolio of *production* automations.

---

## 0. Integration-agnostic by design (HTTP-first)

Every external call — CRM, email platform, Slack, LLM, logging — is a plain **HTTP module** (`http:ActionSendData`) pointed at a documented endpoint, plus Make's universal building blocks (`builtin:BasicRouter`, `builtin:BasicFeeder`, `builtin:Break`, `builtin:Resume`, `gateway:CustomWebHook`).

Why this matters:

- **Imports everywhere.** No dependency on a specific app's module identifier or version — these six module types are stable across every Make account, so a blueprint never imports as "Module Not Found."
- **Vendor-neutral.** Point the CRM modules at HubSpot, Pipedrive, or Attio; the LLM module at OpenAI, Anthropic, or your gateway; the notifier at any Slack/Discord/Teams webhook. Same blueprint.
- **Readable.** One module type, one mental model.

If you prefer a native Make app module (e.g. the official Slack or OpenAI module) in your own copy, swap the HTTP module for it — the data flow is identical. The HTTP form is the portable default.

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

Every blueprint writes a **structured log row** at the end of each run — success *and* failure — by `POST`ing it to a **log sink** endpoint (`https://YOUR_LOG_SINK/append`). Standard schema:

| Field | Example |
|---|---|
| `timestamp` | `2026-06-28T14:03:11Z` |
| `scenario` | `lead-capture-to-crm` |
| `status` | `success` \| `error` \| `skipped` \| `escalated` |
| `record_id` | external id or email |
| `message` | human-readable detail |

The log sink is any HTTP endpoint that accepts that JSON: a **second Make scenario** with a webhook that writes to a Make Data Store, a **Google Sheet** via Apps Script, **Airtable**, or a logging service. [`Utilities/daily-ops-digest`](../Utilities/daily-ops-digest/) reads it back via `GET /summary?since=…`. Full contract: [log-sink.md](log-sink.md). The logging module always carries a `Resume` handler so a logging hiccup never breaks the run.

## 4. Notifications (humans stay in the loop)

- Notifications go out via an **HTTP POST to a Slack incoming webhook** (`https://hooks.slack.com/services/…`) — works in any workspace, no Slack connection or app install needed. Swap the URL for a Discord/Teams webhook and it still works.
- **Failures** after retries are exhausted, and **high-value successes** (a deal created), trigger a notification with scenario name, record id, and detail.
- Notifiers always carry a `Resume` handler — a missed notification never fails the run — and are routed/terminal, never inline in the happy path.

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
