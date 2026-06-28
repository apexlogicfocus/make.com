# Utilities

Reusable infrastructure that supports every other category — observability, dispatch, batching.

| Blueprint | What it does |
|---|---|
| [Daily Ops Digest](daily-ops-digest/) | Reads the shared Automation Log and posts a daily health report to Slack |
| [Scrape YouTube Playlists](scrape-youtube-playlists/) | ⭐ Real exported scenario — paginates a YouTube playlist into a Google Sheet (native YouTube + Sheets modules) |

**Planned:** scenario health watchdog, webhook fan-out dispatcher, rate-limited batcher, Data Store → Sheet exporter, universal error-notifier. See [roadmap](../docs/roadmap.md).
