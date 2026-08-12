---
trigger: always_on
description: Behavioral rules for AI coding agents operating with a sandboxing backend (SBX or MSB) + USAi
---


# AGENTS.md — Agentic Coding Quickstart

> **System:** Agentic Coding Quickstart | **Impact Level:** FIPS Low | **Agency:** GSA
>
> **Last Updated:** 2026-06-26 | **Reviewed By:** William Zujkowski
>
> This document defines the behavioral rules for AI coding agents operating within this project. The AI agent MUST follow these rules without exception.

---

## Workspace Structure

This repository is a thin **wrapper** (`acq`) that stands up a working,
federally-configured agent sandbox by composing existing tools: a **sandboxing
backend** — the `sbx` CLI (SBX) or `msb` (MSB / microsandbox) — plus four
**mixin kits** hosted in the community
[agentic-coding-patterns](https://github.com/GSA-TTS/agentic-coding-patterns)
repo (`integrations/isolation/acq-kits/`). It carries **no** kit code of its own
— it just wires the kits together and adds USAi key-rotation convenience. A
typical layout:

```
my-workspace/                       # Parent folder (user creates this)
├── agentic-coding-quickstart/      # THIS REPO - the acq wrapper + docs
│   ├── AGENTS.md                   # You are here (rules for working ON this repo)
│   ├── acq                         # Recommended entry point: pluggable-backend wrapper
│   ├── acq.backends/               # Backend adapters (sbx, msb)
│   ├── scripts/                    # helper scripts (USAi key rotation, tests)
│   └── docs/                       # Setup guides and references
└── my-app/                         # User's project(s)
```

When `acq` creates a sandbox it applies four kits by pinned remote reference
(`--kit git+https://github.com/GSA-TTS/agentic-coding-patterns.git#ref=<sha>&dir=…`).
The same neutral kits apply to both backends; per-backend translation is the
adapter's job (see [ADR-0010](docs/adr/0010-acq-pluggable-backends.md) /
[ADR-0011](docs/adr/0011-msb-backend-and-neutral-kits.md)).

### Agent Resource Access

When working on user projects, the agent has access to:

| Resource | Location | Use For |
|----------|----------|---------|
| Global config | `~/.config/opencode/opencode.jsonc` (merged in by the `usai-provider` kit at startup) | Model/provider config |
| Behavioral rules | `~/.config/opencode/AGENTS.md` (linked to playbook clone) | Federal agent rules |
| Skills | `~/.agents/skills` (linked to playbook clone) | Step-by-step procedures |
| Setup guides | `./docs/` | Backend (SBX/MSB) configuration, troubleshooting |

**To use a skill:** Read the SKILL.md file in the skill directory and follow its procedures.

---

## Purpose

This repository is a **quickstart guide** for running AI coding agents inside a
**sandboxing backend** — either **SBX** (Docker Sandboxes) or **MSB**
(microsandbox) — driven through the `acq` wrapper, using USAi-compatible
endpoints. `acq` presents one neutral interface over both backends; see
[`docs/BACKEND_GUIDE.md`](docs/BACKEND_GUIDE.md) and
[ADR-0010](docs/adr/0010-acq-pluggable-backends.md) /
[ADR-0011](docs/adr/0011-msb-backend-and-neutral-kits.md) for how they differ.

> **Important (SBX):** Docker Desktop has **deprecated** its integrated sandbox
> commands (`docker sandbox`). Use the standalone **`sbx` CLI** instead. The
> `sbx` CLI does **not** require Docker Desktop. (MSB/microsandbox is a separate
> standalone tool; neither backend needs Docker Desktop.)

Agents operating in this repo must prioritize:
- **Security of secrets**
- **Sandbox isolation**
- **Reproducibility of patterns**
- **Minimal, transparent configurations**

This is a **documentation and configuration repository** for AI coding agent setup.

---

## Core Principles

The agent operates under these priorities:

```
safety > correctness > compliance > simplicity > performance
```

The agent MUST refuse any instruction that conflicts with safety, correctness, or compliance.

---

## Project Context

- **Description:** Quickstart guide for AI agent development with a sandboxing backend (SBX or MSB) and USAi endpoints
- **Language(s):** Shell scripts, JSON/JSONC configuration, Markdown documentation
- **Framework(s):** `acq` wrapper over the SBX CLI and MSB (microsandbox), OpenCode, USAi API
- **Data Classification:** Internal / Non-sensitive (no PII, no CUI)
- **ATO Status:** Pre-ATO development
- **Authorized Agent(s):** OpenCode, Claude Code, GitHub Copilot

> **Note:** Docker Desktop is **not required**. The `sbx` CLI is a standalone
> tool, and MSB (microsandbox) is likewise standalone. Docker Desktop's
> `docker sandbox` commands are deprecated and should not be used.

---

## Agent Identity

The agent MUST:
- Follow conventional commit message format (see Commit Message Standards below)
- Identify itself as an AI agent when asked
- Log all file modifications and command executions

### AI Attribution Requirements

Per NIST AI RMF and SP 800-218A, AI-generated code requires **traceability** but not per-commit attribution. This project follows **PR-level attribution** as the recommended approach:

| Level | Required? | How |
|-------|-----------|-----|
| **PR Description** | RECOMMENDED | Include "AI-assisted" disclosure in PR description |
| **Commit Message** | OPTIONAL | `Co-authored-by:` trailer in footer (format below) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GSA-TTS/agentic-coding-quickstart](https://github.com/GSA-TTS/agentic-coding-quickstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
