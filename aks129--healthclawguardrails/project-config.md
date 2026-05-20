---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A **reference implementation** of security and compliance patterns for AI agent access to
FHIR data via Model Context Protocol (MCP). Version 1.3.0. A [healthclaw.io](https://healthclaw.io) project.

**Supports:**

- **FHIR R4 with US Core v9** profiles — the production standard (stable, widely deployed US healthcare resources). This is what the app primarily does.
- FHIR R6 v6.0.0-ballot3 (experimental ballot resources only: Permission, SubscriptionTopic, DeviceAlert, NutritionIntake)

**Why the `/r6/` route prefix:** The Flask Blueprint and directory are named `r6` from the project's origin as an R6 ballot resource showcase. The actual clinical data pipeline (Conditions, Observations, Immunizations, MedicationRequests, etc.) uses **R4 resources validated against US Core v9 required fields**. The R6 prefix is a route path, not a statement about which FHIR version the clinical resources use.

**What this is:** A pattern library showing how tenant isolation, step-up authorization,
audit trails, PHI redaction, and human-in-the-loop enforcement work together when an
AI agent accesses clinical data through MCP tools.

**What this is NOT:** A production FHIR server. In local mode, resources are stored as
JSON blobs in SQLite. In upstream proxy mode, real FHIR server data flows through the
guardrail stack. Validation is structural only (required fields + value constraints,
no StructureDefinition conformance, no terminology binding).

## Architecture

```text
┌─────────────────────────────────────────────────┐
│  Flask App (Python)                              │
│  ├── /r6/fhir/* — FHIR REST facade (Blueprint)   │
│  ├── /r6/fhir/health — Liveness probe           │
│  ├── /r6/fhir/oauth/* — OAuth 2.1 + SMART       │
│  ├── /fasten/* — Fasten Connect EHR integration  │
│  ├── / — Landing page                            │
│  ├── /skills — Auto-indexed skill catalogue      │
│  ├── /api/subscribe — Resend signup + welcome    │
│  └── /r6-dashboard — Interactive dashboard       │
├─────────────────────────────────────────────────┤
│  MCP Server (Node.js + TypeScript)               │
│  ├── Streamable HTTP (/mcp) — primary transport  │
│  ├── SSE (/sse + /messages) — legacy transport   │
│  ├── HTTP Bridge (/mcp/rpc) — non-MCP clients   │
│  └── Session management + CORS deny-by-default   │
├─────────────────────────────────────────────────┤
│  Data Source (configurable):                     │
│  ├── LOCAL: JSON blobs in SQLite (default)       │
│  └── UPSTREAM: Real FHIR server via httpx proxy  │
│       (HAPI, SMART Health IT, Epic, etc.)        │
│       Guardrails applied to upstream responses   │
├─────────────────────────────────────────────────┤
│  Guardrail Stack (always active):                │
│  ├── PHI redaction on all read paths             │
│  ├── Immutable audit trail                       │
│  ├── Step-up tokens for writes                   │
│  ├── Tenant isolation on every query             │
│  └── URL rewriting (upstream URLs never leak)    │
├─────────────────────────────────────────────────┤
│  Cache: Redis (optional, rate limiting+sessions) │
└─────────────────────────────────────────────────┘
```

### Upstream Proxy Flow

```text
Client → MCP Server → Flask (guardrails) → Upstream FHIR Server
                           ↓
              redaction, audit, step-up,
              tenant isolation, disclaimers,
              URL rewriting
```

## Key Directories

```text
/                         Main Flask app (main.py, app.py, models.py)
/api/                     Vercel serverless entry point (index.py wraps Flask WSGI app)
/r6/                      FHIR Python modules (routes, models, validator, oauth, stepup, audit, redaction, health_compliance, context_builder, rate_limit, fhir_proxy, agent_client, health_context, schema_sync, seed). Named r6/ for historical reasons; handles both R4 US Core and experimental R6 resources.
/r6/fasten/               Fasten Connect EHR integration (routes, models, ingester, verify)
/r6/wearables/            Wearable device sync MCP app (Apple Health / Fitbit poller + UI)
/r6/command_center/       Command Center module (per-tenant ops dashboard)
/services/agent-orchestrator/  Node.js MCP server (TypeScript)
/scripts/                 CLI utilities: import_healthex.py, export_healthex.py, export_healthex_mcp.py (MCP-SDK pull from HealthEx), export_healthex_legacy.py, healthclaw_redact.py (in-process PHI redaction), bot_commands.py (OpenClaw slash-command dispatcher), convert_fasten.py, demo_e2e.sh, smoke_test.py, seed_demo_tenant.py (HTTP + DB seed entry points), seed_openclaw_workspaces.sh, update_agent_prompts.sh, kristy_schedule_watcher.py, kristy_install.sh, build_quickstart_pdf.py
/openclaw/                Telegram bot (bot.py + Dockerfile) — conversational interface to the stack
/hermes/                  Hermes (Nous Research) integration — SOUL persona, MCP config, idempotent install.sh that wires HealthClaw skills into ~/.hermes/. Parallel to /openclaw/; both share the same MCP server.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aks129/HealthClawGuardrails](https://github.com/aks129/HealthClawGuardrails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
