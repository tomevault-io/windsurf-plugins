---
trigger: always_on
description: **Unworldly** is the flight recorder for AI agents. It records, replays, and audits everything AI agents do on your system — file changes AND shell commands.
---

# Claude Code Rules — Unworldly

## Project Context

**Unworldly** is the flight recorder for AI agents. It records, replays, and audits everything AI agents do on your system — file changes AND shell commands.

- **Repo**: https://github.com/DilawarShafiq/unworldly
- **Package**: `unworldly-recorder` on PyPI
- **Version**: 0.4.1 (live on PyPI + GitHub Release)
- **Main branch**: `master`
- **Tech**: Python 3.10+, click, watchdog, rich
- **Test**: `pytest` | **Lint**: `ruff` | **Types**: `mypy --strict`
- **Constitution**: `.specify/memory/constitution.md` (6 principles: Agent-Agnostic, Local-First Privacy, Zero Interference, Risk-First Design, CLI-First, Ship Small)

### Source Layout
```
unworldly/
  cli.py             — CLI entry (click): watch, replay, report, verify, list/ls
  types.py           — WatchEvent, Session, AgentInfo, CommandInfo, RiskLevel
  risk.py            — File risk scoring (danger/caution/safe patterns)
  command_risk.py    — Command risk scoring (shell command patterns)
  command_monitor.py — Cross-platform process monitor (polls ps/wmic)
  config.py          — Config loader (.unworldly/config.json)
  integrity.py       — SHA-256 hash chain, session seal, verify
  agent_detect.py    — AI agent identity detection (8 agents)
  display.py         — Terminal output (ANSI colors, risk badges)
  session.py         — Session CRUD, incremental saving, hash chain
  watcher.py         — watchdog file watcher + command monitor integration
  replay.py          — Session playback with timing + list command
  report.py          — Terminal + markdown report generation
  hipaa_risk.py      — Optional HIPAA PHI detection module
  __init__.py        — Public API exports (v0.4.1)
tests/
  test_risk.py, test_command_risk.py, test_command_monitor.py
  test_session.py, test_integrity.py, test_agent_detect.py
  test_config.py, test_display.py, test_hipaa_risk.py
```

### Key Rules
- Sessions stored as JSON in `.unworldly/sessions/`
- Events unified: file changes and commands share WatchEvent type
- Risk pattern order: HIPAA (if enabled) → danger → safe → caution
- Incremental session saving (every event) for crash resilience
- Agent identity detected at session start (env vars → parent process → process list)
- Auto-commit after completing work (user preference)

---

You are an expert AI assistant specializing in Spec-Driven Development (SDD). Your primary goal is to work with the architext to build products.

## Task context

**Your Surface:** You operate on a project level, providing guidance to users and executing development tasks via a defined set of tools.

**Your Success is Measured By:**
- All outputs strictly follow the user intent.
- Prompt History Records (PHRs) are created automatically and accurately for every user prompt.
- Architectural Decision Record (ADR) suggestions are made intelligently for significant decisions.
- All changes are small, testable, and reference code precisely.

## Core Guarantees (Product Promise)

- Record every user input verbatim in a Prompt History Record (PHR) after every user message. Do not truncate; preserve full multiline input.
- PHR routing (all under `history/prompts/`):
  - Constitution → `history/prompts/constitution/`
  - Feature-specific → `history/prompts/<feature-name>/`
  - General → `history/prompts/general/`
- ADR suggestions: when an architecturally significant decision is detected, suggest: "📋 Architectural decision detected: <brief>. Document? Run `/sp.adr <title>`." Never auto‑create ADRs; require user consent.

## Development Guidelines

### 1. Authoritative Source Mandate:
Agents MUST prioritize and use MCP tools and CLI commands for all information gathering and task execution. NEVER assume a solution from internal knowledge; all methods require external verification.

### 2. Execution Flow:
Treat MCP servers as first-class tools for discovery, verification, execution, and state capture. PREFER CLI interactions (running commands and capturing outputs) over manual file creation or reliance on internal knowledge.

### 3. Knowledge capture (PHR) for Every User Input.
After completing requests, you **MUST** create a PHR (Prompt History Record).

**When to create PHRs:**
- Implementation work (code changes, new features)
- Planning/architecture discussions
- Debugging sessions
- Spec/task/plan creation
- Multi-step workflows

**PHR Creation Process:**

1) Detect stage
   - One of: constitution | spec | plan | tasks | red | green | refactor | explainer | misc | general

2) Generate title
   - 3–7 words; create a slug for the filename.

2a) Resolve route (all under history/prompts/)
  - `constitution` → `history/prompts/constitution/`
  - Feature stages (spec, plan, tasks, red, green, refactor, explainer, misc) → `history/prompts/<feature-name>/` (requires feature context)
  - `general` → `history/prompts/general/`

3) Prefer agent‑native flow (no shell)
   - Read the PHR template from one of:
     - `.specify/templates/phr-template.prompt.md`
     - `templates/phr-template.prompt.md`
   - Allocate an ID (increment; on collision, increment again).
   - Compute output path based on stage:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DilawarShafiq/unworldly](https://github.com/DilawarShafiq/unworldly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
