# CRM

Automations that keep your contact database clean, complete, and current. The defining pattern here is **dedup-before-write** — never create a duplicate record.

| Blueprint | What it does |
|---|---|
| [Lead Capture → CRM](lead-capture-to-crm/) | Form submission → validated, de-duplicated contact + Slack ping |
| [CRM Contact Enrichment](crm-contact-enrichment/) | Scheduled batch enrichment with write-back and idempotency |

**Planned:** duplicate sweep & merge, territory routing, bi-directional sheet sync, lifecycle automation. See [roadmap](../docs/roadmap.md).
