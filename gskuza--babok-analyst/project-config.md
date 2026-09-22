---
trigger: always_on
description: Generates all 8 stages with quality checks (up to 3 iterations per stage) before final human approval.
---

# Copilot Instructions for BABOK Analyst

Developer guide for working on the BABOK Analyst repository. This is a multi-interface business analysis platform implementing the BABOK v3 framework as a 9-stage pipeline (Stage 0 charter + Stages 1–8).

**Current Version:** 2.3.0 | **Node.js:** 18+ required | **Module System:** ESM throughout (`"type": "module"`)

---

## Build, Test & Lint Commands

### Root Workspace Tests

```bash
npm install                       # Install root dependencies
npm test                          # All tests: unit + integration + plugin/hook/uninstall
npm run check-versions            # Verify version sync across all package.json files
npm run sync-codex-plugin         # Regenerate .codex-plugin from .claude-plugin
```

**Run individual test files:**
```bash
node --test tests/unit/project.test.js
node --test tests/unit/journal.test.js
node --test tests/unit/two-key-gate.test.js
node --test tests/unit/scoring.test.js
node --test tests/unit/validation.test.js
node --test tests/unit/templates.test.js
node --test tests/integration/cli-workflow.test.js
node --test tests/plugin-manifest.test.cjs
node --test tests/hooks.test.cjs
```

### CLI (`cli/`)

```bash
cd cli
npm install
npm link                          # Register 'babok' command globally (for development)
npm test                          # Smoke test: babok --help
node bin/babok.js --help          # Run without npm link
```

### MCP Server (`babok-mcp/`)

```bash
cd babok-mcp
npm install
npm run dev                       # Watch mode (node --watch bin/babok-mcp.js)
npm test                          # Smoke test (runs src/test/smoke.js)
npm start                         # Production mode
```

### Web UI (`web/`)

```bash
cd web
npm install
npm run dev                       # Development server (http://localhost:3000)
npm run build                     # TypeScript check + production build
npm run lint                      # ESLint check
```

---

## High-Level Architecture

### Four Interfaces, One Storage Layer

BABOK Analyst ships as four independent interfaces that all read/write the same canonical storage:

| Interface | Purpose | Entry Point | Storage Access |
|-----------|---------|------------|-----------------|
| **CLI** | Terminal-based workflows | `cli/bin/bakok.js` | Direct file I/O + journal management |
| **MCP Server** | Model Context Protocol for Claude/GPT | `babok-mcp/src/server.js` | 19 tools + 9 stage resources |
| **Web UI** | Dashboard & project browser | `web/app/` (Next.js App Router) | REST API + server-side readers |
| **Plugin** | VS Code / Claude Code / Copilot integration | `commands/*.md` + `hooks/*.cjs` | Delegates to CLI or MCP |

**Canonical Storage:** `projects/<project_id>/` (not `BABOK_Analysis/`, which is legacy)
- `PROJECT_JOURNAL_<id>.json` — State machine (stage status, approvals, decisions, assumptions)
- `STAGE_0N_<name>.md` — Per-stage deliverable markdown files
- `.stage_N.lock` — File lock for team collaboration (2-hour stale threshold)

### 9-Stage Pipeline (default profile `babok`)

The stage shape is declared per **pipeline profile** in `profiles/<id>/profile.json` (schema `profiles/profile.schema.json`). The default `babok` profile points at the existing files below; the `consulting` profile (`profiles/consulting/`, prefix `BC-`, stages 0–6) covers non-IT advisory engagements. The profile is chosen at creation (`babok new --profile`, `babok_new_project { profile }`, `/babok-new-consulting`) and stored in `journal.profile`; everything else derives from the journal. Loader `cli/src/profiles.js` is mirrored byte-for-byte in `babok-mcp/src/lib/profiles.js` (`tests/unit/lib-parity.test.js`).

Each stage represents a distinct business analysis deliverable:

| Stage | Deliverable | File |
|-------|------------|------|
| 0 | Project Charter (Go/No-Go gate) | `STAGE_00_Project_Charter.md` |
| 1 | Stakeholder Mapping & Success Criteria | `STAGE_01_Project_Initialization.md` |
| 2 | AS-IS Process Analysis | `STAGE_02_Current_State_Analysis.md` |
| 3 | Problem Domain & Root Cause Analysis | `STAGE_03_Problem_Domain_Analysis.md` |
| 4 | Requirements (FR/NFR, MoSCoW, RTM) | `STAGE_04_Solution_Requirements.md` |
| 5 | TO-BE Design & Future State | `STAGE_05_Future_State_Design.md` |
| 6 | Gap Analysis & Implementation Roadmap | `STAGE_06_Gap_Analysis_Roadmap.md` |
| 7 | Risk Assessment & Mitigation | `STAGE_07_Risk_Assessment.md` |
| 8 | Business Case & ROI Model | `STAGE_08_Business_Case_ROI.md` |

**Stage Lifecycle:** `not_started → in_progress → completed → approved | rejected`

Stages are loaded from `BABOK_AGENT/stages/BABOK_agent_stage_N.md` at runtime — no build step, changes take effect immediately.

### Two-Key Journal: Agent/Human Separation of Duties

Stage approval is enforced **outside the LLM** as a hard gate:

1. **Agent** calls `babok_save_deliverable`, then `babok_submit_for_review` (writes `PROJECT_JOURNAL.json`)
2. **Human** runs `babok approve <id> <stage>` — this is the **only path** that sets `status: approved`
3. **To revise** an approved stage, either side calls `babok_open_revision` first (resets to `in_progress`)

**Enforcement:** `hooks/babok-gate.cjs` (PreToolUse hook)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GSkuza/BABOK_ANALYST](https://github.com/GSkuza/BABOK_ANALYST) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
