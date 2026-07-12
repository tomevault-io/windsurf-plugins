---
trigger: always_on
description: This repo is **the source for the ODF Agent Team**, an OpenCode skill/agent pack for
---

# ODF Agent Team — AGENTS.md

This repo is **the source for the ODF Agent Team**, an OpenCode skill/agent pack for
structured Odoo development. It is **not** an Odoo project itself. It installs into `~/.config/opencode/` via `install.sh` and is used from Odoo worktrees.

---

## ODF Overview — What It Does

ODF (Odoo Development Framework) is a spec-driven pipeline for building Odoo modules:

```
init → preflight → assess → qa-plan → design → implement → verify → archived
```

Each phase is a sub-agent with a defined input/output contract. The orchestrator never writes code — it delegates everything, tracks state, and gates progression.

### What you can do

| Command | Purpose |
|---------|---------|
| `/odf-new <name>` | Start a new change: preflight → full pipeline |
| `/odf-continue [name]` | Resume a change from its last completed phase |
| `/odf-status [name]` | Show change state, artifacts, task progress |
| `/odf-explore <topic>` | Investigate Odoo patterns before committing to a change |
| `/odf-fix <description>` | Lightweight 3-step bugfix: diagnose → fix → verify |
| `/odf-agent-new <description>` | Create custom Odoo sub-agents from natural language |
| `/odf-profile switch` | Toggle between `default` (r1 for ASSESS/VERIFY, kimi for others) and `cheap` (kimi for all) |
| `/odf-registry-refresh` | Pick up skill/agent changes after install |
| `/odf-tdd on/off` | Toggle strict test-before-code enforcement |
| `/odf-health` | Verify installation and project detection |

### Pipeline phases

| Phase | Agent | Output | Gate |
|-------|-------|--------|------|
| **ASSESS** | `odoo_functional_consultant` | Strategy (standard vs custom) + functional spec | `question` tool — proceed/adjust/cancel |
| **QA-PLAN** | `odoo_qa_engineer` | Test plan, scenarios, coverage targets | `question` tool — approve/adjust |
| **DESIGN** | `odoo_backend_engineer` / `odoo_frontend_engineer` | Technical design + tasks | `question` tool — approve & implement |
| **IMPLEMENT** | Agent by task domain | Code + tests, in batches | `question` tool — continue/adjust |
| **VERIFY** | `odoo_qa_engineer` | Test run, lint, spec compliance | PASS or FAIL |
| **ARCHIVED** | (auto) | Retrospective saved to Engram | — |

---

## Architecture

```
                   ┌──────────────────────┐
                   │   odoo_orchestrator   │  ← NEVER writes code, only delegates
                   │   (AGENT)             │
                   └──────┬───────┬───────┘
                          │       │
              ┌───────────┘       └───────────┐
              ▼                               ▼
   ┌──────────────────────┐       ┌──────────────────────┐
   │   odf-delegation.ts  │       │   12 sub-agents       │
   │   (PLUGIN)           │──────▶│   (via task())        │
   │                      │       │                       │
   │   Registry cache     │       │  odoo_backend_engineer │
   │   Profile resolution │       │  odoo_frontend_enginee │
   │   Skill injection    │       │  odoo_qa_engineer      │
   │   Status resolver    │       │  odoo_functional_con.  │
   │   Community tools    │       │  odoo_code_reviewer    │
   └──────────────────────┘       │  odoo_api_integrator   │
                                  │  odoo_context_gatherer │
                                  │  odoo_skill_finder     │
                                  │  odoo_dba_devops       │
                                  │  odoo_stock_lot_spec.  │
                                  │  odoo_upgrade_migrator │
                                  └──────────────────────┘
```

### Plugin tools (`odf-delegation.ts`)

10 tools injected at runtime into the orchestrator's tool list:

| Tool | Purpose |
|------|---------|
| `odf_delegate` | Route phase + prompt to the right sub-agent with skill injection |
| `odf_skill_inject` | Match and inject relevant skill compact rules into a prompt |
| `odf_skill_resolve` | Preview which skills match a task WITHOUT executing |
| `odf_registry_read` | Read and cache `odf-registry.json` with TTL and file watcher |
| `odf_profile_select` | Preview which model profile applies to a phase |
| `odf_notebooklm_lookup` | Search NotebookLM notebooks for reference material |
| `odf_community_tool_detect` | Check if a community tool (e.g., CodeGraph) is available |
| `odf_community_tool_install` | Install and wire a community tool |
| `odf_status` | Resolve ODF change status from Engram observations |

### Skills system

31 skills covering:

- **OCA governance**: PR workflow, maturity levels, commit messages, contribution guidelines
- **OCA style**: Python, XML, JavaScript/CSS standards
- **Odoo patterns**: field types, computed fields, inheritance, constraints, domains, onchange, context/env, security, data migration, API integration, migration guides, debug patterns
- **ODF phases**: assess, design, implement, verify, fix, TDD, QA, exploration, agent builder, chained PR
- **Shared conventions**: Engram persistence, result contract, skill resolver, Odoo source paths
- **ODF onboarding**: guided walkthrough of the full pipeline on a real codebase

Max 5 skills per delegation via compact rules.

### Model profiles

| Profile | ASSESS | VERIFY | Other phases |
|---------|--------|--------|--------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antoniodavid/odf-agent-team](https://github.com/antoniodavid/odf-agent-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
