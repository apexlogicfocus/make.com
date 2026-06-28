# Contributing

Thanks for contributing a blueprint! Every scenario in this library meets the same bar. Use this checklist before opening a PR.

## Blueprint PR checklist

- [ ] **Folder** named `kebab-case` under the right category (`CRM/`, `Sales/`, …).
- [ ] Folder contains `blueprint.json` + `README.md` (and optionally `diagram.md`).
- [ ] **Sanitized**: no API keys, tokens, webhook URLs, connection ids, or real customer data. (`grep -iE "token|secret|api[-_]?key|bearer" blueprint.json` returns nothing.)
- [ ] **Retries**: every external API module has an `onerror` → `Break` handler with sensible `count`/`interval`.
- [ ] **Logging**: a structured log row is written on success *and* failure paths.
- [ ] **Notifications**: failures (post-retry) notify a channel; routed, not inline.
- [ ] **Module IDs** sequential from 1; designer coords laid out left→right.
- [ ] **README** documents: what it does, trigger, apps/connections needed, setup table, test payload, and the data flow.
- [ ] Imports cleanly into a fresh Make scenario (test it).

See [docs/conventions.md](docs/conventions.md) for the full spec and rationale.

## README template for a blueprint

```markdown
# <Scenario Name>

> One-line value statement.

## What it does
…

## Trigger
Webhook | Schedule | Watch — details.

## Apps & connections
| App | Used for | Scope |

## Setup
| Setting | Where | Example |

## Test payload
\`\`\`bash
curl …
\`\`\`

## Data flow
(mermaid or bullet list)

## Error handling & logging
…
```

## Reusing third-party blueprints

If any part derives from an MIT-licensed source, keep that source's copyright
notice in the blueprint folder (e.g. a `NOTICE` file) alongside this repo's MIT license.
