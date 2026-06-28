# Daily Ops Digest

> One Slack message every morning telling you how every other automation in this library performed overnight — successes, errors, escalations.

## What it does

This is the **observability layer** for the whole library. Every blueprint POSTs its run rows to a shared [log sink](../../docs/log-sink.md); this scenario calls the sink's `GET /summary` once a day, reads the counts by `status` over the last 24 hours, and posts a digest to a Slack webhook. If there were any errors, it says so; if not, it reports all systems healthy.

It's also the **simplest blueprint to test first** — it only needs a Slack incoming webhook and a reachable log sink.

## Trigger

**Scheduled** — daily, first thing in the morning.

## Apps & connections

| App | Module | Used for |
|---|---|---|
| HTTP | GET log-sink summary | Pull success/error/escalated counts (24h) |
| HTTP | POST to Slack webhook | Post the digest |

## Setup

| Setting | Where | Example |
|---|---|---|
| Log sink | Module 1 → `url` | `https://YOUR_LOG_SINK/summary` ([setup](../../docs/log-sink.md)) |
| Window | Module 1 `qs.since` | `addDays(now; -1)` (24h) |
| Slack webhook | Module 2 → `url` | `https://hooks.slack.com/services/…` |
| Schedule | Scenario scheduling | daily 08:30 |

> The summary endpoint should return `{ "success": <n>, "error": <n>, "escalated": <n> }`. See the [log sink contract](../../docs/log-sink.md).

## Data flow

```mermaid
flowchart LR
    S[GET /summary?since=-24h] --> M[Build digest] --> SL[POST Slack webhook]
```

## Why this matters for a portfolio

Most automation demos have no observability — you only find out something broke when a customer complains. This library is designed so that **every scenario logs to one place** and this digest surfaces it. It's the difference between "I built some automations" and "I run an automation *system*."

## Error handling & logging

- The summary fetch `Break`-retries briefly (2× / 3 min).
- The Slack post `Break`-retries 3× — the digest *is* the deliverable.
- An `if()` in the message switches between "check the log" and "all healthy" based on the error count.
