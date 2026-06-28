# Examples

Test data and versioned AI prompts for the blueprints.

## Payloads

Sample JSON bodies for the webhook-triggered scenarios. POST one to a blueprint's webhook URL to test it.

| File | For blueprint |
|---|---|
| [`payloads/lead.json`](payloads/lead.json) | CRM → Lead Capture |
| [`payloads/booking.json`](payloads/booking.json) | Sales → Meeting Booked → Deal |
| [`payloads/ticket.json`](payloads/ticket.json) | AI Agents → Support Triage |
| [`payloads/new-hire.json`](payloads/new-hire.json) | HR → New-Hire Onboarding |

Example:
```bash
curl -X POST "https://hook.us1.make.com/YOUR_WEBHOOK_ID" \
  -H "Content-Type: application/json" \
  --data @examples/payloads/lead.json
```

## Prompts

The LLM prompts for the `AI Agents/` blueprints live here, versioned separately so you can tune behavior without re-importing a scenario.

| File | For blueprint |
|---|---|
| [`prompts/support-triage.md`](prompts/support-triage.md) | Support Triage |
| [`prompts/content-repurposer.md`](prompts/content-repurposer.md) | Content Repurposer |
