---
trigger: always_on
description: > **This file is the persistent memory and operating manual for this repository.**
---

# CLAUDE.md — VIGILANCE T5.3 Agentic Wrapper Framework

> **This file is the persistent memory and operating manual for this repository.**
> Update it whenever architecture changes, schemas evolve, or milestone status shifts.
> Last updated: July 2026 — reflects actual implemented state of the repository. Schemas frozen with T5.4 (GFT); C4 verb catalogue documented; M6 closed; T5.5 interface question resolved at July 1 KOM (T5.5 is blueprint/scenario collection, not policy enforcement — downstream consumer of `t53.policy_updates` now open).

---

## Project Identity

**Project:** VIGILANCE
**EU Grant:** Horizon Europe — GAP-101249737
**Duration:** 36 months
**INNOV role:** Core technical contributor and task lead for T5.3

**Work Package:** WP5 — Agentic AI Cybersecurity Platform
**Task:** T5.3 — Agentic Wrappers for Cybersecurity Technologies
**Task Lead:** INNOV-ACTS

**One-line purpose:** T5.3 is the operational execution bridge between WP5 AI intelligence (agents, orchestration, knowledge) and the real cybersecurity tools deployed in the pilot environments. It normalises raw events into a canonical format, routes them through safety checks, executes AI-approved actions via vendor-specific adapters, and records every step for audit.

**Primary deliverable contribution:** D5.1 (Framework Architecture and Data Models)

### Pilots in scope for INNOV

| Pilot | Task | Organisation | Country | Sector |
|---|---|---|---|---|
| Pilot #1 | T6.3 | OTE | Greece | Telecom SOC |
| Pilot #4 | T6.5 | Siemens | Romania | Industry 4.0 / Manufacturing |

> ⚠️ **HARD CONSTRAINT:** Port of Rotterdam (Pilot #2 / T6.4, Netherlands) and CaixaBank (Pilot #3 / T6.6, Spain) are **not** in scope for INNOV. Never reference these pilots in INNOV-produced documents, diagrams, or code. The T5.3 framework defines sector profiles for all four GA pilots because the GA mandates transferable wrappers — but INNOV validates only against OTE and Siemens.

---

## Implementation Status

**The repository contains a fully implemented, containerised, running service.** This is not a design-phase repository. All core components are implemented and tested.

```
vigilance-GATE/
│
├── CLAUDE.md                          ← this file (persistent memory)
├── BLUEPRINT.md                       ← architectural blueprint (design rationale, GA mandate mapping)
├── Dockerfile                         ← python:3.11-slim image
├── docker-compose.yml                 ← full stack: gate + rabbitmq + ollama + dozzle
├── pyproject.toml                     ← package manifest and dependencies
│
├── data/                              ← generated output (mounted from container /app/data)
│   └── workflow_audit.csv             ← one row per pipeline execution (raw → canonical → result)
│
├── vigilance/                         ← main application package
│   ├── main.py                        ← entrypoint (REST API + broker consumer)
│   ├── service.py                     ← service lifecycle / broker consumer
│   ├── pipeline.py                    ← T53Pipeline — INTEGRATED mode orchestration
│   ├── workflow_logger.py             ← WorkflowCSVLogger — per-execution audit CSV
│   ├── api/                           ← REST API (FastAPI, port 8000)
│   │   └── app.py                     ← POST /api/v1/events, POST /api/v1/action-requests, GET /api/v1/health, GET /api/v1/profiles
│   ├── broker/                        ← RabbitMQ broker (pika); InMemoryBroker for tests
│   ├── llm/                           ← LLM abstraction layer
│   │   ├── base.py                    ← LLMProvider ABC + StubLLMProvider
│   │   └── ollama_provider.py         ← OllamaLLMProvider (Mistral 7B + Nemo 12B)
│   ├── models/                        ← Pydantic v2 data models (frozen schema)
│   │   ├── canonical_event.py
│   │   ├── action_request.py
│   │   ├── execution_result.py
│   │   ├── guardrail_check.py
│   │   └── audit_record.py
│   └── components/
│       ├── c1_ingestion/              ← C1: Normalizer + 5 parsers (CEF, ECS, OT JSON, Syslog, LLM)
│       ├── c3_execution/              ← C3: ActionExecutor + PolicyTranslator (NL→Rego, implemented)
│       ├── c4_adapters/               ← C4: ToolAdapter ABC + 12 plugins across 4 sectors
│       │   ├── telecom/               ← ote_siem, ote_iam, ote_ids
│       │   ├── industry4/             ← scada_opcua, ot_iam, industrial_siem
│       │   ├── maritime/              ← port_siem, port_iam, port_ops
│       │   └── finance/               ← bank_siem, bank_iam, fraud_engine
│       ├── c5_safety/                 ← C5: SafetyGate + AuditLog
│       └── c6_profiles/               ← C6: ProfileManager + SectorProfile dataclass
│
├── profiles/                          ← sector profile YAMLs
│   ├── telecom.yaml                   ← OTE / TELECOM (confidence_threshold: 0.80)
│   ├── industry4.yaml                 ← Siemens / INDUSTRY_4 (ot_safety_flag: true)
│   ├── maritime.yaml                  ← Port of Rotterdam / MARITIME (GA transferability)
│   └── finance.yaml                   ← CaixaBank / FINANCE (confidence_threshold: 0.85)
│
├── schemas/                           ← data model and broker schemas
│   ├── README.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtouloup/vigilance-GATE](https://github.com/mtouloup/vigilance-GATE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
