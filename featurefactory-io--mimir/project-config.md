---
trigger: always_on
description: Mimir is a **self-evolving engineering playbook platform**. It lets teams document, evolve, and leverage software development methodologies through a Django web UI and an MCP (Model Context Protocol) server that AI assistants query directly.
---

# CLAUDE.md — Mimir Development Guide

## What Is Mimir

Mimir is a **self-evolving engineering playbook platform**. It lets teams document, evolve, and leverage software development methodologies through a Django web UI and an MCP (Model Context Protocol) server that AI assistants query directly.

**Two-part system:**
- **FOB (this repo)** — Django 5 app with SQLite, runs MCP server + web server concurrently
- **HOMEBASE** (separate repo) — Django + HTMX, eventing → Neo4j graph DB, aggregates PIPs from FOBs

**Key innovation:** AI assistants can propose playbook improvements (PIPs) that engineers review and approve.

**Current Status:** Production-ready MVP with full CRUDLF for Playbooks, Workflows, Activities, Agents, Skills, Phases (as modeled), Artifacts (partial), and **Rules** (playbook-scoped, Activity M2M). MCP integration includes 50+ tools (including rule CRUDLF and `set_activity_rules`).

---

## Architecture

### Directory Layout

```
mimir/                        # Django project config (settings, urls, wsgi)
methodology/                  # Core Django app (models, services, views, templates)
  models/                     # Playbook, Workflow, Phase, Activity, Artifact, Skill, Agent, Rule, …
  services/                   # Business logic (shared by UI and MCP)
  views/                      # Thin Django view controllers
  utils/                      # markdown_renderer.py (Markdown + Mermaid)
mcp_integration/              # MCP server (thin wrappers over services)
  tools.py                    # MCP tool definitions (playbooks, workflows, activities, skills, rules, …)
  context.py                  # User context
  management/commands/mcp_server.py
accounts/                     # User auth
tests/
  unit/                       # Model and service unit tests
  integration/                # MCP tools, multi-model workflows
  e2e/                        # Playwright browser tests
docs/
  features/act-*/             # BDD Gherkin specs (source of truth for features)
  architecture/SAO.md         # System architecture overview (read before big changes)
  plans/                      # Per-feature implementation plans
  ux/                         # Design system, IA guidelines, user journey
.windsurf/
  rules/                      # 38 development convention rules
  workflows/FeatureFactory/   # Complete product development playbook (6 workflows)
    ESM/                      # Envision the System (7 activities)
    DTA/                      # Define Architecture (18 activities)
    DSP/                      # Deploy Software Process (6 activities)
    EST/                      # Estimate the Project (8 activities)
    BSP/                      # Bootstrap Project (8 activities)
    BPE/                      # Build Feature (8 activities)
```

### Data Model Hierarchy

```
Playbook (status: draft/released/disabled; version: 0.x → 1.0+)
  └─ Workflow (name, abbreviation, order)
       ├─ Phase (optional; currently string field, not separate model)
       └─ Activity (name, guidance: Markdown+Mermaid, order)
            └─ Artifact (input/output, producer/consumer)
Playbook also has: Agent, Skill, **Rule** (IDE-style rules; many-to-many with Activities)
```

**Implementation status:**
| Entity | Status |
|--------|--------|
| Playbook | ✅ Full CRUDLF |
| Workflow | ✅ Full CRUDLF + export/import |
| Activity | ✅ Full CRUDLF |
| Phase | ⚠️ String field only |
| Artifact | ⚠️ Model exists, partial |
| Agent | ✅ Full CRUDLF (Act 7) |
| Skill | ✅ Full CRUDLF (Act 8) |
| Rule | ✅ Full CRUDLF + export to `rules/` |
| PIP | ❌ Not built in FOB UI (V2 / Homebase narrative retained in specs) |

### Services Pattern

Services in `methodology/services/` are **shared by both UI views and MCP tools** — never add MCP-specific logic to services. MCP tools only add: user context lookup, permission check, serialization.

Version auto-increment (Activity update → increments grandparent Playbook version) happens in the MCP tool wrapper, not the service.

### MCP Access Rules

- **Draft playbooks** (v0.x): Full CRUD via MCP
- **Released playbooks** (v≥1.0): Read-only via MCP; changes must go through PIPs

---

## Terminology

| Internal Code | User-Facing UI |
|---------------|----------------|
| `methodology` (app, models, services) | "playbooks" |
| `MethodologyRepository` | — |
| `manage.py mcp_server` | — |

Always use "playbooks" in UI text; always use `methodology` in code.

---

## FeatureFactory Workflows

Mimir development follows the **FeatureFactory v3.8** playbook — a complete product development methodology from idea to shipped feature.

### Inception Phase (run once per project)

1. **ESM — Envision the System** (7 activities)
   - User journey, screen flows, feature files (BDD), IA guidelines, mockups
   - Output: `docs/features/**`, `docs/ux/screen-flow.drawio`

2. **DTA — Define Architecture** (18 activities)
   - Technical decisions across 16 domains → System Architecture Overview
   - Output: `docs/architecture/SAO.md`

3. **DSP — Deploy Software Process** (6 activities)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FeatureFactory-io/mimir](https://github.com/FeatureFactory-io/mimir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
