---
trigger: always_on
description: nWave is an AI-powered workflow framework that orchestrates specialized Claude AI agents through disciplined software development waves. It runs inside Claude Code, enforcing TDD, phase tracking, and deterministic validation at every step.
---

# CLAUDE.md — nWave Developer Experience System

## What is nWave?

nWave is an AI-powered workflow framework that orchestrates specialized Claude AI agents through disciplined software development waves. It runs inside Claude Code, enforcing TDD, phase tracking, and deterministic validation at every step.

**Core mission**: Replace ad-hoc AI coding with a structured, auditable, wave-based methodology — from discovery to deployment.

**Two packages**:
- `nwave` (this repo, private) — development, full source, CI/CD
- `nwave-ai` (public, PyPI via `nwave-ai/nwave` repo) — installer CLI for end users

---

## Project Structure

```
nWave-dev/
├── nWave/                    # Framework definition (agents, commands, skills, templates)
│   ├── agents/               # 23 agent specifications (YAML frontmatter + markdown)
│   ├── tasks/nw/             # 21 slash command definitions (/nw-deliver, /nw-design, etc.)
│   ├── skills/               # 98 agent skill files (deep domain knowledge)
│   ├── templates/            # Methodology templates (TDD schema, pre-commit, README)
│   ├── data/                 # Configuration data, methodologies, research references
│   ├── hooks/                # Agent lifecycle hooks
│   ├── framework-catalog.yaml  # Central metadata registry (agents, commands, quality gates)
│   └── VERSION               # Framework version (synced from pyproject.toml)
│
├── src/des/                  # DES runtime (Deterministic Execution System)
│   ├── domain/               # Business logic (phase events, turn counter, timeout, policies)
│   ├── application/          # Use cases (orchestrator, validators, services)
│   ├── ports/                # Interfaces (driver + driven ports)
│   └── adapters/             # Implementations (hooks, filesystem, config, logging, git)
│
├── nwave_ai/                 # Public CLI package (thin wrapper)
│   └── cli.py                # Entry: install, uninstall, version commands
│
├── scripts/
│   ├── install/              # Installation pipeline
│   │   ├── plugins/          # Plugin system (agents, commands, DES, skills, templates, utilities)
│   │   ├── install_nwave.py  # Main installer orchestrator
│   │   ├── preflight_checker.py
│   │   └── installation_verifier.py
│   ├── hooks/                # Pre-commit hook scripts (all Python, zero shell)
│   ├── framework/            # Build utilities (sync names, create tarballs, docgen)
│   ├── validation/           # YAML & frontmatter validators
│   └── build_dist.py         # Distribution builder
│
├── tests/                    # 5-layer test suite
│   ├── des/                  # DES tests (unit/, integration/, acceptance/, e2e/)
│   ├── installer/            # Installer tests (unit/, acceptance/, e2e/)
│   ├── plugins/              # Plugin system tests
│   ├── bugs/                 # Regression tests
│   ├── build/                # Build script tests
│   └── conftest.py           # Root fixtures, auto-marking by directory
│
├── docs/
│   ├── guides/               # Tutorials and how-tos (public)
│   ├── reference/            # Auto-generated API/command reference (public)
│   ├── architecture/         # ADRs, design decisions (public)
│   └── analysis/             # Internal analysis (EXCLUDED from public sync)
│
├── .github/workflows/
│   ├── ci.yml                # 4-stage CI (lint → validate → test → sync)
│   └── release.yml           # 5-job release (bump → build → release → sync → pypi)
│
└── pyproject.toml            # Single source of truth for versions and tool config
```

---

## Architecture: DES (Deterministic Execution System)

DES follows **hexagonal architecture** (ports & adapters):

```
Claude Code Hooks (pre-tool-use, subagent-stop, post-tool-use)
        │
        ▼
┌─ Adapters (drivers) ──────────────────────────────────────┐
│  claude_code_hook_adapter.py  →  JSON hook protocol        │
└────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ Application Layer ───────────────────────────────────────┐
│  DESOrchestrator       — prompt rendering, phase execution │
│  PreToolUseService     — validates before Agent invocation │
│  SubagentStopService   — validates after sub-agent returns │
│  TemplateValidator     — checks 9 mandatory sections       │
│  StaleExecutionDetector — detects abandoned phases         │
└────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ Domain Layer ────────────────────────────────────────────┐
│  PhaseEvent, TurnCounter, TimeoutMonitor, TDDSchema       │
│  DES enforcement policies, Result<T,E> types              │
└────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ Ports (driven) ──────────────────────────────────────────┐
│  FileSystemPort, ConfigPort, TimeProvider, AuditLogWriter  │
│  LoggingPort, TaskInvocationPort                           │
└────────────────────────────────────────────────────────────┘
        │
        ▼
┌─ Adapters (driven) ───────────────────────────────────────┐
│  RealFileSystem, EnvironmentConfigAdapter, SystemTimeProvider│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nWave-ai/nWave](https://github.com/nWave-ai/nWave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
