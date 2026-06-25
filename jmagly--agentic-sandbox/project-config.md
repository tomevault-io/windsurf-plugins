---
trigger: always_on
description: Project configuration for AI assistance platforms.
---

# AGENTS.md

Project configuration for AI assistance platforms.

## Project Overview

Agentic Sandbox is a runtime isolation platform for persistent, unrestricted AI agent processes. It provides QEMU/KVM VMs with secure isolation, shared storage via virtiofs, task orchestration, and a web-based management dashboard.

## Tech Stack

- **Language**: Rust, Python, Shell
- **Runtime**: Rust toolchain (cargo), Python 3.8+
- **Framework**: gRPC API, HTTP REST API, WebSocket streaming
- **Test Framework**: `cargo test`
- **Package Manager**: Cargo, Python (setuptools/pyproject)

## Development Commands

| Command | Description |
|---------|-------------|
| `make build` | Build Rust components in `management/`, `agent-rs/`, and `cli/`. |
| `make test` | Run Rust unit tests (default target) |
| `make test-e2e` | Run end-to-end tests via `scripts/run-e2e-tests.sh` |
| `make lint` | Check Rust formatting (cargo fmt) |
| `make check` | Run Rust format check + unit tests |
| `cd management && ./dev.sh` | Start management server in dev mode |
| `cd management && cargo build --release` | Build Rust management server |
| `cd agent-rs && cargo build --release` | Build Rust agent client |

## Project Structure

```
agent-rs/          → Agent client (Rust)
agents/            → Agent artifacts/config
api/               → API definitions and handlers
cli/               → CLI (Rust)
configs/           → Configuration files
deploy/            → Deployment assets
docs/              → Documentation
images/            → VM/base image assets
management/        → Management server (Rust)
proto/             → gRPC protobufs
runtimes/          → Runtime definitions
scripts/           → Utility scripts and systemd units
tests/             → Test data and E2E documentation; Rust E2E tests live under management/tests/e2e_*
```

## Detected Project Files

| Artifact | Location | Notes |
|---------|----------|-------|
| Go modules | None | Go components removed |
| Rust crates | `management/Cargo.toml`, `agent-rs/Cargo.toml`, `cli/Cargo.toml` | Management server, agent client, CLI |
| Build system | `Makefile` | Primary build/test targets |

## CI/CD Signals

No `.github/workflows/` directory detected in the repository root.
---

## Team Directives

<!-- PRESERVED SECTION -->

Team directives:
- Issue tracking standard: use Gitea issues for backlog and task tracking.
- Rules reference: AIWG rules index lives at `.codex/rules/RULES-INDEX.md` (load full rule via the @-links when needed).

<!-- /PRESERVED SECTION -->

---

## AIWG Framework

This project uses the AIWG SDLC framework for software development workflows.

**Framework Registry:** `.aiwg/frameworks/registry.json`
- **Registry Version:** 1.0.0
- **Installed Frameworks:** sdlc-complete v1.0.0 (healthy)

### AIWG State Detection

| Directory | Status | Notes |
|----------|--------|-------|
| `.factory/droids/` | Not detected | Factory AI agents not present |
| `.opencode/agent/` | Not detected | OpenCode agents not present |
| `.cursor/rules/` | Not detected | Cursor rules not present |
| `.codex/agents/` | Detected | Codex agents present |
| `.aiwg/` | Detected | AIWG artifacts present |

### Available Agents

**Architecture & Design:**
- `architecture-designer` - System architecture and technical decisions
- `database-architect` - Database design and optimization
- `api-designer` - RESTful/GraphQL API design

**Development & Implementation:**
- `software-implementer` - Test-first development (TDD)
- `integrator` - Component integration and build management
- `test-engineer` - Comprehensive test suite creation
- `test-architect` - Test strategy and framework design

**Quality & Security:**
- `code-reviewer` - Code quality and security review
- `security-architect` - Security design and threat modeling
- `qa-engineer` - Quality assurance and testing
- `performance-engineer` - Performance optimization

**Operations & Release:**
- `devops-engineer` - CI/CD and deployment automation
- `sre` - Site reliability and monitoring
- `release-manager` - Release coordination and rollout

**Management & Coordination:**
- `project-manager` - Project planning and tracking
- `product-owner` - Product vision and requirements
- `scrum-master` - Agile process facilitation
- `business-analyst` - Requirements gathering and analysis

**Specialized:**
- `technical-writer` - Documentation and content
- `ux-designer` - User experience and interface design
- `data-engineer` - Data pipelines and analytics

**Deployment:** Agents are deployed to platform-specific directories:
- Claude Code: `.claude/agents/`
- GitHub Copilot: `.github/agents/`
- Factory AI: `.factory/droids/`
- Cursor: `.cursor/rules/`
- OpenCode: `.opencode/agent/`
- Codex: `.codex/agents/`

**Detected in this repo:** `.codex/agents/`

**Full catalog:** `@~/.local/share/ai-writing-guide/agentic/code/frameworks/sdlc-complete/agents/`

### Workflow Commands

**Intake & Planning:**
- `intake-wizard` - Generate project intake forms
- `intake-from-codebase` - Analyze codebase to generate intake
- `project-status` - Check current project phase
- `project-health-check` - Comprehensive health assessment

**Phase Transitions:**
- `flow-inception-to-elaboration` - Transition to Elaboration
- `flow-elaboration-to-construction` - Transition to Construction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmagly/agentic-sandbox](https://github.com/jmagly/agentic-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
