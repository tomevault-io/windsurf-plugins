---
trigger: always_on
description: Custom fork of [AWS AI-DLC](https://github.com/awslabs/aidlc-workflows) (v0.2.0).
---

# AIDLC Development Repo

Custom fork of [AWS AI-DLC](https://github.com/awslabs/aidlc-workflows) (v0.2.0).
Extends upstream with a multi-agent orchestrator, skills enforcement, hallucination
prevention, CodeGraph integration, and persistent memory (Engram).

> This repo IS the AIDLC toolchain itself. Modifying it means modifying the workflow engine,
> not running it against an application codebase.

---

## Directory map

| Path | Purpose |
|------|---------|
| `aidlc-rules/aws-aidlc-rules/core-workflow.md` | Stage workflow rules |
| `aidlc-rules/aws-aidlc-rule-details/` | Detailed stage rule files (inception / construction / operations / extensions) |
| `.github/agents/orchestrator.agent.md` | Multi-agent orchestrator (custom agent) |
| `.github/agents/stage/` | 13 stage subagents (`*.agent.md`) |
| `.github/agents/cross-cutting/` | conflict-resolver, knowledge-agent |
| `.github/agents/custom/` | User-defined custom agents |
| `.github/prompts/` | User-invocable `/factory-*` prompt commands |
| `.github/skills/` | Custom skills copied here on `--tool copilot` install |
| `.agents/custom-skills/` | Fork-specific skills (source of truth in this repo) |
| `.claude/commands/factory-*.md` | Slash command definitions (Claude Code / OpenCode) |
| `.aidlc-orchestrator/runtime/` | Runtime architecture docs |
| `.aidlc-orchestrator/contracts/` | JSON Schema handoff contracts for every stage I/O |
| `.aidlc-orchestrator/budgets/default.yaml` | Per-stage model assignments |
| `aidlc-scripts/factory_*.py` | Runtime scripts (run manager, conflict, merge-reviews, validate, telemetry) |
| `aidlc-scripts/install_aidlc.py` | Installer — copies rules + agents into target projects |
| `aidlc-docs/` | Generated artifacts from AIDLC runs |
| `src/` | Source library (memory store, adapters) |
| `tests/` | Test suite |
| `docs/` | Supporting documentation (WORKING-WITH-AIDLC, TROUBLESHOOTING) |

---

## Multi-agent orchestrator (GitHub Copilot)

The orchestrator routes development requests through specialized stage agents using
JSON Schema handoff contracts. Invoke from Copilot Chat in **Agent mode** by typing `/`
and selecting a prompt from `.github/prompts/`.

| Prompt / Phase | What it does |
|-----------------|-------------|
| `/factory-spec` | Workspace scout + requirements analysis (Phase 0) |
| `/factory-plan` | Execution plan + optional unit decomposition (Phase 1) |
| `/factory-build` | Parallel code-gen + build/test with file-glob locks (Phase 5) |
| `/factory-review` | Reviewer pool: code / security / performance / simplifier |
| `/factory-ship` | Release notes, ADRs, CHANGELOG, CI/CD wiring |
| `/factory-resume` | Resume interrupted run from last checkpoint |
| `/factory-replay` | Re-run from a specific stage |
| `/factory-state` | Show run status, stage, budget |
| `/factory-help` | Full command reference |

Agent definitions: `.github/agents/**/*.agent.md`. Runtime spec: `.aidlc-orchestrator/runtime/index.md`.

---

## Key rules when modifying this repo

- **Contracts are the interface**: stage agent I/O defined by JSON Schema in `.aidlc-orchestrator/contracts/`. Changes to agent outputs must update contracts.
- **Rule files are source of truth**: `aidlc-rules/aws-aidlc-rule-details/` is the corpus. Duplicate logic is a bug.
- **Installer is the distribution mechanism**: new orchestrator files must be wired into `aidlc-scripts/install_aidlc.py`.
- **Runtime state is gitignored**: `.aidlc-orchestrator/runs/`, `.aidlc-orchestrator/knowledge/`, `.codegraph/` — never commit these.
- **Parity**: body content of stage agents must stay in sync across `.claude/`, `.cursor/`, `.opencode/`, and `.github/agents/` (Copilot uses `*.agent.md` + platform-specific frontmatter).

---

## VS Code / Copilot setup

`.vscode/settings.json` enables nested subagent spawning and registers AIDLC file locations:

```json
{
  "chat.subagents.allowInvocationsFromSubagents": true,
  "chat.agentFilesLocations": { ".github/agents": true },
  "chat.promptFilesLocations": { ".github/prompts": true },
  "chat.instructionsFilesLocations": { ".github/instructions": true }
}
```

The orchestrator custom agent declares an `agents:` allowlist and includes the `agent` tool — both are required for subagent delegation.

---

## Copilot execution constraints

Copilot differs from Claude Code in important ways:

### 1. Sequential-only agent calls

Copilot processes one `agent` tool call at a time. The orchestrator MUST:
- Invoke agents sequentially (one at a time, wait for result, then proceed)
- Never attempt to invoke multiple agents in a single response
- Use the `agent` tool only — `Task(subagent_type=...)` is Claude Code syntax and will fail

### 2. Spawn budget

Keep total `agent` invocations per command under 8 to avoid context exhaustion:

| Command | Max spawns | Strategy |
|---------|-----------|----------|
| `factory-spec` | 3 | workspace-scout + analyst ×2 (or inline scout for greenfield) |
| `factory-plan` | 2 | workflow-planner + unit-decomposer (skip story-writer unless requested) |
| `factory-build` | 4 units × 3 spawns = 12 max | Cap at 4 units per run; use `merged_plan_generate` for SMALL tier |
| `factory-review` | 1–2 | Default to `reviewer-code` only; add others only when manifest specifies |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mbg999/aidlc-factory](https://github.com/Mbg999/aidlc-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
