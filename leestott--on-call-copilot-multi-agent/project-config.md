---
trigger: always_on
description: This document describes the multi-agent architecture of On-Call Copilot: how the agents are orchestrated, what each specialist does, and how to extend or customize them.
---

# Agents

This document describes the multi-agent architecture of On-Call Copilot: how the agents are orchestrated, what each specialist does, and how to extend or customize them.

---

## Overview

On-Call Copilot uses **four specialist agents** running **concurrently** via `ConcurrentBuilder` from the Microsoft Agent Framework. Each agent receives the full incident payload, processes it through Microsoft Foundry Model Router, and returns a JSON fragment covering its designated output keys. The orchestrator merges all fragments into a single unified response.

```
                         Incident JSON
                              │
                              ▼
                    ┌───────────────────┐
                    │  OncallCopilotAgent│
                    │  (orchestrator)    │
                    └────────┬──────────┘
                             │
              asyncio.gather() — all 4 run in parallel
           ┌─────────┬──────┴──────┬──────────┐
           ▼         ▼            ▼          ▼
      ┌─────────┐ ┌─────────┐ ┌────────┐ ┌────────┐
      │ Triage  │ │ Summary │ │ Comms  │ │  PIR   │
      │ Agent   │ │ Agent   │ │ Agent  │ │ Agent  │
      └────┬────┘ └────┬────┘ └───┬────┘ └───┬────┘
           │           │          │           │
           └───────────┴──────────┴───────────┘
                              │
                     Merge JSON fragments
                              │
                              ▼
                    Structured JSON response
```

All agents share a single **Model Router** deployment (`AZURE_OPENAI_CHAT_DEPLOYMENT_NAME=model-router`). Model Router automatically routes each request to the best model based on prompt complexity — no model-selection logic is needed in agent code.

---

## Agent Reference

### Triage Agent

| Property | Value |
|----------|-------|
| **Name** | `triage-agent` |
| **File** | [`app/agents/triage.py`](app/agents/triage.py) |
| **Role** | Root cause analysis, immediate actions, missing data identification, runbook alignment |

**Output keys:**

| Key | Type | Description |
|-----|------|-------------|
| `suspected_root_causes` | array | Each entry has `hypothesis` (string), `evidence` (string array), `confidence` (0–1 float) |
| `immediate_actions` | array | Each entry has `step` (string), `owner_role` (string), `priority` (P0–P3) |
| `missing_information` | array | Each entry has `question` (string), `why_it_matters` (string) |
| `runbook_alignment` | object | `matched_steps` (string array), `gaps` (string array) |

**Guardrails:**
- Secrets are redacted as `[REDACTED]`
- Insufficient data → `confidence: 0` and `missing_information` populated (no hallucination)
- Sparse incidents get diagnostic steps in `immediate_actions` rather than remediation

---

### Summary Agent

| Property | Value |
|----------|-------|
| **Name** | `summary-agent` |
| **File** | [`app/agents/summary.py`](app/agents/summary.py) |
| **Role** | Concise incident narrative for SRE teams |

**Output keys:**

| Key | Type | Description |
|-----|------|-------------|
| `summary.what_happened` | string | 2–4 sentence factual summary: trigger event, affected services, failure mode, scope |
| `summary.current_status` | string | Prefixed with `ONGOING` / `MITIGATED` / `MONITORING` / `RESOLVED` plus brief detail |

**Behaviour:**
- Presence of `timeframe.end` → resolved status
- No `end` timestamp → ongoing unless other signals indicate otherwise

---

### Comms Agent

| Property | Value |
|----------|-------|
| **Name** | `comms-agent` |
| **File** | [`app/agents/comms.py`](app/agents/comms.py) |
| **Role** | Audience-appropriate incident communications |

**Output keys:**

| Key | Type | Description |
|-----|------|-------------|
| `comms.slack_update` | string | Slack-formatted message with emoji, severity, status, impact, next steps |
| `comms.stakeholder_update` | string | Non-technical executive summary: business impact, customer effect, resolution status |

**Slack emoji conventions:**

| Condition | Emoji |
|-----------|-------|
| Active SEV1/SEV2 | `:rotating_light:` |
| Degraded | `:warning:` |
| Resolved | `:white_check_mark:` |

**Stakeholder update rules:**
- No jargon or unexplained acronyms
- Focus on customer experience and business impact
- Blameless tone

---

### PIR Agent

| Property | Value |
|----------|-------|
| **Name** | `pir-agent` |
| **File** | [`app/agents/pir.py`](app/agents/pir.py) |
| **Role** | Post-incident report: timeline reconstruction, customer impact, prevention actions |

**Output keys:**

| Key | Type | Description |
|-----|------|-------------|
| `post_incident_report.timeline` | array | Each entry has `time` (HH:MMZ or ISO) and `event` (string), chronologically ordered |
| `post_incident_report.customer_impact` | string | Quantified impact: users affected, error rates, revenue estimates |
| `post_incident_report.prevention_actions` | array | Specific, actionable measures with suggested owner roles |

**Behaviour:**
- Timeline is reconstructed from `alerts`, `logs`, and `metrics` timestamps
- Ongoing incidents end with `{"time": "ONGOING", "event": "..."}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leestott/On-Call-Copilot-Multi-Agent](https://github.com/leestott/On-Call-Copilot-Multi-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
