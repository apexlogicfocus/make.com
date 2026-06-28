# Roadmap — scaling to 50+ blueprints

Status legend: ✅ shipped · 🛠️ in progress · 📋 planned

## Foundation
- ✅ Repo structure, conventions, shared patterns
- ✅ Universal skeleton + per-category architecture diagrams
- ✅ Getting-started + contributing guides

## CRM (target: 8)
- ✅ Lead Capture → CRM (dedup + create/update)
- ✅ CRM Contact Enrichment (API lookup + write-back)
- 📋 Duplicate sweep & merge
- 📋 Lead routing by territory/score
- 📋 Bi-directional CRM ↔ Sheet sync
- 📋 Lifecycle stage automation
- 📋 Inactive-contact archival
- 📋 Form → multi-CRM fan-out

## Sales (target: 8)
- ✅ Meeting Booked → Deal
- ✅ Stale Deal Reminder
- 📋 Quote/proposal generator
- 📋 Won-deal → onboarding handoff
- 📋 Lost-deal reason capture
- 📋 Pipeline daily snapshot
- 📋 Lead-to-SDR round-robin
- 📋 Renewal reminder

## Marketing (target: 7)
- ✅ Newsletter Signup Sync
- 📋 Multi-channel content distributor
- 📋 UTM → attribution logger
- 📋 Webinar registration → reminders
- 📋 Lead-magnet delivery
- 📋 Re-engagement drip trigger
- 📋 Ad-lead → CRM + alert

## AI Agents (target: 8)
- ✅ AI Support Ticket Triage (classify + route)
- ✅ AI Content Repurposer (1 → many formats)
- 📋 RAG-backed FAQ responder
- 📋 Lead scoring agent
- 📋 Call/transcript summarizer
- 📋 Email draft assistant
- 📋 Sentiment monitor
- 📋 Document data extractor

## Finance (target: 6)
- ✅ Overdue Invoice Dunning (staged)
- 📋 Payment received → reconcile
- 📋 Expense receipt router
- 📋 MRR/ARR weekly report
- 📋 Subscription churn alert
- 📋 Invoice generation from deal

## HR (target: 6)
- ✅ New-Hire Onboarding (fan-out)
- 📋 Offboarding deprovision
- 📋 PTO request approval
- 📋 Document collection chaser
- 📋 Birthday/anniversary recognition
- 📋 Candidate pipeline sync

## Utilities (target: 7)
- ✅ Daily Ops Digest
- ✅ Scrape YouTube Playlists (real export — pagination → Google Sheets)
- 📋 Scenario health-check watchdog
- 📋 Webhook fan-out dispatcher
- 📋 Rate-limited batch processor
- 📋 Data Store → Sheet exporter
- 📋 Universal error-notifier
- 📋 Scheduled backup of Data Stores

## Cross-cutting
- 📋 Demo GIFs per category
- 📋 Import screencasts
- 📋 Blueprint validator script (CI)
