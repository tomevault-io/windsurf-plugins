---
trigger: always_on
description: Pinscope validates hardware schematics against component datasheets. It extracts constraints from PDFs, parses netlists and BOMs into a queryable graph, and runs an agentic validation loop to flag design violations.
---

# Pinscope — Agentic Schematic Validation

Pinscope validates hardware schematics against component datasheets. It extracts constraints from PDFs, parses netlists and BOMs into a queryable graph, and runs an agentic validation loop to flag design violations.

> **Open-core note.** This is the open-source core. A small set of files are
> "gateway-owned seams" — pass-through stubs here (`frontend/src/proxy.ts`,
> `use-optional-auth.ts`, `clerk-theme-provider.tsx`,
> `components/billing/*`, `sidebar-auth.tsx`, `pricing-section.tsx`,
> `analytics/*`, `lib/csp-hosts.ts`) that the hosted-cloud repo replaces
> with auth/billing implementations. Keep their export signatures stable,
> and never import auth/billing SDKs anywhere else in the frontend. On the
> backend, everything reaches billing only through
> `backend/services/billing_hook.py:get_billing()` (a no-op here).

## System Overview

Three layers:

| Layer | Location | Purpose |
|-------|----------|---------|
| **Core library** | `backend/pinscopex/` | Models, parsers, graph builder, agentic validator, passive resolver, taxonomy, BOM summary, derating |
| **Backend** | `backend/` | FastAPI app — async pipeline orchestration, SSE progress, project/file storage |
| **Frontend** | `frontend/` | Next.js 16 app — project dashboard, pipeline progress, report viewer, derating, admin dashboard |

Plus `skills/` — Claude Console Skills for datasheet extraction (pintable, patterns, specs).

The pipeline stages: Parse BOM → Extract IC Pintables → Extract Simple Components → Extract Passives → DigiKey Auto-Resolve + Value Fallback → Build Graph → Direct Datasheet Review. Pipeline runs can be cancelled mid-execution via `POST /api/pipeline/{id}/cancel`.

## Example Project

`simple_project/` is the reference design for development and testing:

- **MCU**: TI MSPM0G3507SPTR (U3) — 48-pin LQFP
- **USB-UART Bridge**: CH340E (U2)
- **LDO Regulator**: SPX3819M5-L-3-3 (U1) — 5V to 3.3V
- **ESD Protection**: USBLC6-2SC6 (D1)
- **Crystal**: 8 MHz (X1) with 18pF load caps (C9, C10)

Files: `.asc` (PADS-PCB netlist; `.edn` EDIF 2.0.0 also accepted), `.csv`/`.xlsx` (BOM), `design_graph.json` (committed reference fixture used by tests).

## Architecture Principles

- **Modular extractors** — Domain-specific extraction per component type, unified constraint schema
- **Netlist as graph** — Queryable bipartite graph (components + nets) with traversal helpers
- **Claude API for PDF extraction** — Forced tool calls for structured output (pintable, passive patterns, specs)
- **Prompt caching** — Extraction and review API calls use `cache_control={"type": "ephemeral"}` on system prompts and input context to reduce cost on repeated calls
- **Claude Console Skills** — Extraction prompts deployed as managed skills; skill_ids and versions loaded from `backend/skills_manifest.json` (upload your own via `scripts/upload_skills.py`)
- **Direct datasheet review** — Claude reads the IC datasheet PDF and circuit neighborhood together, compares to reference application circuit, and flags issues via graph query tools (`find_connected_components`, `get_net_for_pin`, `get_pintable`)
- **Datasheet page trimming** — Large PDFs are keyword-trimmed to relevant pages before sending to Claude, reducing token cost (`pypdf`)
- **DigiKey fallback (exact MPN only)** — When pattern-based and direct extraction fail, DigiKey API fetches product parameters for auto-resolve. DigiKey matches only on exact MPN; fuzzy hits are rejected to avoid polluting the shared library with wrong-dielectric / wrong-voltage parts.
- **Value-string fallback** — When DigiKey misses an R/C/L/FB passive, a value-string resolver maps the BOM `Value` string to typed passive specs. Value-derived specs are persisted per-project only — never to the shared library.
- **Per-IC review error isolation** — Direct datasheet review runs each IC independently; one malformed payload or bad response cannot kill the whole run. Failed ICs surface as skipped components with the error.
- **Cross-IC excerpt budget (per-neighbor)** — To verify an interface finding the reviewer can pull a *connected* IC's datasheet pages (`get_datasheet_excerpt`). The budget is a global per-review page ceiling **plus a per-neighbor sub-budget**, so verifying one interface is never starved by pages already spent on other neighbors.
- **Finding normalization is downgrade-only** — A post-review per-IC normalize pass (`services/normalize_findings.py`) drops self-cancelling findings, merges same-root-cause findings, and re-grades severity — but only ever *downward*. A deterministic clamp caps each finding at the reviewer's calibrated severity (and any `Unverified:` finding at WARNING, preserving the prefix).
- **Cross-IC finding dedup** — After all per-IC reviews complete, a single pass (`services/dedupe_findings.py`) collapses one physical interface defect reported from both endpoints into a single finding. Gated by `cross_ic_dedup_enabled`; fail-soft.
- **Capacitor voltage derating** — Deterministic derating table computed from graph (ceramic/tantalum/electrolytic percentages, pass/fail per capacitor)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Faradworks/Pinscope](https://github.com/Faradworks/Pinscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
