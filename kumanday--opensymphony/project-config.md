---
trigger: always_on
description: This file provides persistent context for AI agents working on this repository.
---

# AGENTS.md

This file provides persistent context for AI agents working on this repository.

## Project Overview

<!-- Describe your project here. What does it do? What problem does it solve? -->

## Technology Stack

<!-- List your technologies: languages, frameworks, databases, etc. -->

- Language: <!-- e.g., Python 3.11, TypeScript 5.0 -->
- Framework: <!-- e.g., FastAPI, React, Next.js -->
- Database: <!-- e.g., PostgreSQL, SQLite -->
- Testing: <!-- e.g., pytest, vitest -->

## Coding Standards

### General

- Keep functions small and focused
- Write self-documenting code with clear names
- Add comments only for "why", not "what"
- Follow existing patterns in the codebase

### Formatting

<!-- Add your formatting commands -->

- Format command: `<!-- e.g., make format, npm run format -->`
- Lint command: `<!-- e.g., make lint, npm run lint -->`
- Type check: `<!-- e.g., make typecheck, npm run typecheck -->`

### Testing

<!-- Add your testing requirements -->

- Test command: `<!-- e.g., make test, npm test -->`
- Coverage requirement: <!-- e.g., 80%, 100% for critical paths -->
- Test location: `<!-- e.g., tests/, __tests__/ -->`

## Project Structure

```
<!-- Customize this structure for your project -->
.
├── src/                    # Source code
├── tests/                  # Test files
├── docs/                   # Documentation
├── configs/                # Configuration files
├── scripts/                # Utility scripts
├── AGENTS.md               # This file
├── WORKFLOW.md             # OpenSymphony configuration
└── README.md               # Project readme
```

## Key Directories

<!-- Document important directories -->

- `src/` - <!-- Main source code -->
- `tests/` - <!-- Test files -->
- `docs/` - <!-- Documentation -->

## Dependencies

### Runtime

<!-- List key runtime dependencies -->

### Development

<!-- List key dev dependencies -->

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `EXAMPLE_VAR` | <!-- Description --> | Yes/No |

## Local Development Setup

<!-- Steps to set up local development -->

```bash
# Example setup steps
# 1. Install dependencies
# 2. Configure environment
# 3. Run tests
```

## PR Requirements

Before submitting a PR:

1. All tests pass
2. Code is formatted
3. Lint checks pass
4. New code has tests
5. Documentation updated if needed

## Architecture Decisions

<!-- Document key architecture decisions -->

### Decision 1

- **Context**: <!-- Why this decision was needed -->
- **Decision**: <!-- What was decided -->
- **Consequences**: <!-- Impact and trade-offs -->

## Known Issues / Gotchas

<!-- Document any quirks or known issues -->

## References

<!-- Links to relevant external documentation -->

- [Framework Docs](https://example.com)
- [API Reference](https://example.com/api)

## Preserved Existing AGENTS.md

The following content was preserved from the repository's previous `AGENTS.md` during `opensymphony init`.

# AGENTS.md

## Mission

Build OpenSymphony as a Rust implementation of the Symphony service specification using OpenHands agent-server for execution and FrankenTUI for the optional terminal UI.

This repository is an orchestrator. It is not a chat app, not a general workflow engine, and not a thin wrapper around OpenHands.

## Authority order

When sources disagree, use this order:

1. upstream `openai/symphony` `SPEC.md`
2. pinned OpenHands SDK agent-server docs and the wire-contract notes in `docs/websocket-runtime.md`
3. this repository's `docs/`
4. the task file currently being implemented
5. local code comments and tests

Do not silently invent behavior when the upstream spec or chosen integration contract is explicit.

## Hard invariants

### Orchestration

- The Rust orchestrator is the sole authority over scheduling state.
- Workers report events and outcomes to the orchestrator.
- No background task may mutate scheduling state except through orchestrator-owned commands or messages.
- Tracker polling remains required even though agent runtime updates use WebSockets.

### Workspace safety

- Every issue maps to exactly one sanitized workspace key.
- Workspace paths must remain inside the configured workspace root.
- The agent runtime must execute with `cwd == issue_workspace_path`.
- Never run agent code in the orchestrator repository root, temp root, or an unsanitized path.

### OpenHands integration

- Target the SDK agent-server HTTP and WebSocket contract.
- Do not implement against `openhands serve`.
- Do not implement against the web-app Socket.IO protocol.
- Operations are REST. Runtime streaming is WebSocket.
- The WebSocket readiness barrier is the first `ConversationStateUpdateEvent`.
- Always reconcile events after WebSocket readiness and after reconnect.
- One OpenHands conversation is reused per issue by default.
- A fresh conversation gets the full workflow prompt. A resumed conversation gets continuation guidance only.
- Local MVP uses one local agent-server subprocess shared across issues, not one server per issue.
- Local MVP does not require Docker per workspace.

### Tracker contract

- The orchestrator reads Linear directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kumanday/OpenSymphony](https://github.com/kumanday/OpenSymphony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
