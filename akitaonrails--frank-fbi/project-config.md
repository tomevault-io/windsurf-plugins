---
trigger: always_on
description: Frank FBI is a headless Rails 8.1 email fraud/spam detection system. It receives forwarded suspicious emails via Gmail IMAP, analyzes them through a 6-layer pipeline, and replies with a score and explanation report. Also supports a lighter 3-layer messenger triage pipeline for WhatsApp/Telegram/Signal screenshots.
---

# CLAUDE.md — Frank FBI (Fraud Bureau of Investigation)

## Project Summary

Frank FBI is a headless Rails 8.1 email fraud/spam detection system. It receives forwarded suspicious emails via Gmail IMAP, analyzes them through a 6-layer pipeline, and replies with a score and explanation report. Also supports a lighter 3-layer messenger triage pipeline for WhatsApp/Telegram/Signal screenshots.

## Tech Stack

- Ruby 4.0.1, Rails 8.1.2 (API-only)
- SQLite3 for all databases (primary, queue, cache, cable)
- Solid Queue for background jobs
- ruby_llm gem for LLM calls via OpenRouter
- ferrum gem for headless Chrome screenshots
- Action Mailbox for inbound email processing
- Docker Compose for deployment (4 services)

## Commands

```bash
# Run tests
bin/rails test

# Run smoke test (known spam .eml through deterministic layers)
bin/rails frank_fbi:smoke_test

# Analyze a specific .eml file
bin/rails "frank_fbi:analyze_eml[path/to/file.eml,submitter@email.com]"

# Triage a messenger screenshot .eml
bin/rails "frank_fbi:triage_eml[path/to/file.eml,submitter@email.com]"

# Start the background worker
bin/jobs

# Start the IMAP mail fetcher
bin/rails frank_fbi:fetch_mail

# Start the web server
bin/rails server

# Manage allowed senders
bin/rails "frank_fbi:add_sender[user@example.com]"
bin/rails "frank_fbi:add_senders[a@example.com,b@example.com]"
bin/rails "frank_fbi:remove_sender[user@example.com]"
bin/rails frank_fbi:list_senders

# Build and push Docker image to Gitea registry
bin/deploy
```

## Architecture

### Fraud Analysis Pipeline (6 Layers)

1. **Header Auth** (weight 0.15) — SPF/DKIM/DMARC/ARC checks, Reply-To mismatch, antispam headers. Fully deterministic.
2. **Sender Reputation** (weight 0.15) — WHOIS domain age, DNSBL blacklists, local reputation DB. Depends on Layer 1 (sender IP).
3. **Content Analysis** (weight 0.15) — Pattern matching for urgency, financial fraud, PII requests, authority impersonation, URL shorteners, dangerous attachments. Fully deterministic.
4. **External API** (weight 0.15) — VirusTotal and URLhaus URL scanning. Depends on Layer 3 (extracted URLs).
5. **Entity Verification** (weight 0.10) — OSINT-based sender identity verification via Brave Search, domain checks, web presence validation. Depends on Layers 1+3. After completion, triggers ScreenshotCaptureJob to capture website screenshots of reference links.
6. **LLM Analysis** (weight 0.30) — 3 parallel LLM consultations via OpenRouter (Claude, GPT-4o, Grok) with consensus building. Depends on Layers 1-5.

Final score: `sum(layer_score * weight * confidence) / sum(weight * confidence)`

### Messenger Triage Pipeline (3 Layers)

1. **URL Scan** (weight 0.40) — VirusTotal + URLhaus scanning
2. **File Scan** (weight 0.30) — VirusTotal file scanning
3. **LLM Triage** (weight 0.30) — 3 parallel LLM consultations for messenger scam patterns

### Community Threat Intelligence Reporting

After report delivery, `CommunityReportingJob` submits IOCs from high-confidence fraudulent emails (score >= 85, verdict "fraudulent") to community threat intel databases. Best-effort — errors are logged but never block the pipeline. All API keys are optional; missing keys silently skip that provider.

Providers: ThreatFox (abuse.ch), AbuseIPDB, SpamCop. IOCs extracted: URLs, domains, sender IPs, file hashes. Freemail domains, cloud IPs, and confirmed-clean URLs are filtered out.

Audit trail stored in `CommunityReport` (one per email, idempotent).

### Screenshot Capture

After entity verification finds reference links, ScreenshotCaptureJob captures headless Chrome screenshots (via ferrum) in parallel with LLM analysis. The pipeline waits for screenshots to complete before generating the report. Screenshots are resized to 560px, JPEG quality 60, embedded as base64 in the HTML report. Errors are caught gracefully — report renders without screenshots if capture fails.

### Job Flow (Fraud)

`EmailParsingJob` → Layers 1+3 (parallel) → Layers 2+4+5 (after dependencies) → ScreenshotCaptureJob (after Layer 5) → Layer 6 → `ScoreAggregationJob` → `ReportGenerationJob` → `ReportDeliveryJob` → `CommunityReportingJob` (best-effort)

Orchestrated by `Analysis::PipelineOrchestrator` — each job calls `advance(email)` after completion.

### Data Model

- `Email` — central record, has_many analysis_layers/llm_verdicts, has_one analysis_report/community_report. `pipeline_type` field: "fraud_analysis" (default) or "messenger_triage"
- `AnalysisLayer` — one per layer per email (6 for fraud, 3 for triage), unique on [email_id, layer_name]
- `LlmVerdict` — one per LLM provider per email (3 per email), unique on [email_id, provider]
- `KnownDomain` — domain reputation cache (WHOIS, DNSBL, fraud ratio)
- `KnownSender` — sender reputation tracking, belongs_to KnownDomain
- `UrlScanResult` — VirusTotal/URLhaus cache with TTL, unique on [url, source]
- `AnalysisReport` — rendered HTML/text report per email

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akitaonrails/frank_fbi](https://github.com/akitaonrails/frank_fbi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
