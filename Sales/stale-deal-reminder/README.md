# Stale Deal Reminder

> Find open deals that have gone quiet and nudge the owner before the deal goes cold — a daily pipeline-hygiene robot.

## What it does

On a daily schedule, it searches for open deals with **no activity in 7+ days**, and for each one creates a follow-up task due tomorrow, notifies the owner's Slack channel, and logs the nudge. Because it runs once a day, each deal is nudged at most once per day.

## Trigger

**Scheduled** — recommended daily at the start of the work day.

## Apps & connections

| App | Module | Used for |
|---|---|---|
| HTTP | Search deals | Find idle open deals |
| Iterator | Basic feeder | Loop the results |
| HTTP | Create task | Put a follow-up on the owner's list |
| HTTP | POST to Slack webhook | Notify the owner's channel |
| HTTP | POST to log sink | Audit log |

## Setup

| Setting | Where | Example |
|---|---|---|
| Idle threshold | Module 1 `lastActivityBefore` | `addDays(now; -7)` |
| Task due date | Module 3 `dueDate` | tomorrow |
| Owner Slack id | `2.ownerSlackId` | map from your CRM |
| Schedule | Scenario scheduling | daily 08:00 |

## Data flow

```mermaid
flowchart LR
    Q[Search idle deals] --> IT[Iterator]
    IT --> T[Create follow-up task]
    T --> DM[Slack DM owner]
    DM --> L[(Log)]
```

## Error handling & logging

- Search + task creation `Break`-retry 3× / 5 min.
- Slack `Resume`s — a failed DM still leaves the task + log in place.
- Scheduled once daily, so each deal is nudged at most once per day; the log row records the deal id and owner for traceability. (Want hard idempotency on manual re-runs? Have your log sink upsert by `<dealId>-<date>`.)
