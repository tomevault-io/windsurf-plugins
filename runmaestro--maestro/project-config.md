---
trigger: always_on
description: Essential guidance for working with this codebase. For detailed architecture, see [ARCHITECTURE.md](ARCHITECTURE.md). For development setup and processes, see [CONTRIBUTING.md](CONTRIBUTING.md).
---

# CLAUDE.md

Essential guidance for working with this codebase. For detailed architecture, see [ARCHITECTURE.md](ARCHITECTURE.md). For development setup and processes, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Documentation Index

This guide has been split into focused sub-documents for progressive disclosure:

| Document                             | Description                                                                                                 |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| [[CLAUDE-PATTERNS.md]]               | Core implementation patterns (process management, settings, modals, themes, Auto Run, SSH, Encore Features) |
| [[CLAUDE-IPC.md]]                    | IPC API surface (`window.maestro.*` namespaces)                                                             |
| [[CLAUDE-PERFORMANCE.md]]            | Performance best practices (React optimization, debouncing, batching)                                       |
| [[CLAUDE-WIZARD.md]]                 | Onboarding Wizard, Inline Wizard, and Tour System                                                           |
| [[CLAUDE-FEATURES.md]]               | Usage Dashboard and Document Graph features                                                                 |
| [[CLAUDE-AGENTS.md]]                 | Supported agents and capabilities                                                                           |
| [[CLAUDE-SESSION.md]]                | Session interface (agent data model) and code conventions                                                   |
| [[CLAUDE-PLATFORM.md]]               | Cross-platform concerns (Windows, Linux, macOS, SSH remote)                                                 |
| [AGENT_SUPPORT.md](AGENT_SUPPORT.md) | Detailed agent integration guide                                                                            |

---

## Agent Behavioral Guidelines

Core behaviors for effective collaboration. Failures here cause the most rework.

### Surface Assumptions Early

Before implementing non-trivial work, explicitly state assumptions. Never silently fill in ambiguous requirements—the most common failure mode is guessing wrong and running with it. Format: "Assumptions: 1) X, 2) Y. Correct me now or I proceed."

### Manage Confusion Actively

When encountering inconsistencies, conflicting requirements, or unclear specs: **STOP**. Name the specific confusion, present the tradeoff, and wait for resolution. Bad: silently picking one interpretation. Good: "I see X in file A but Y in file B—which takes precedence?"

### Push Back When Warranted

Not a yes-machine. When an approach has clear problems: point out the issue directly, explain the concrete downside, propose an alternative, then accept the decision if overridden. Sycophancy ("Of course!") followed by implementing a bad idea helps no one.

### Enforce Simplicity

Natural tendency is to overcomplicate—actively resist. Before finishing: Can this be fewer lines? Are abstractions earning their complexity? Would a senior dev say "why didn't you just..."? Prefer the boring, obvious solution.

### Maintain Scope Discipline

Touch only what's asked. Do NOT: remove comments you don't understand, "clean up" orthogonal code, refactor adjacent systems as side effects, or delete seemingly-unused code without approval. Surgical precision, not unsolicited renovation.

### Dead Code Hygiene

After refactoring: identify now-unreachable code, list it explicitly, ask "Should I remove these now-unused elements: [list]?" Don't leave corpses. Don't delete without asking.

### Validate Before Push

Before pushing any branch, re-run the relevant formatting, lint, type-check, and test commands for the changes you made. Fix any issues those commands surface, include the fixes in the branch, and only then push or update the PR.

---

## Standardized Vernacular

Use these terms consistently in code, comments, and documentation:

### Terminology: Agent vs Session

In Maestro, the terms "agent" and "session" have distinct meanings:

- **Agent** - An entity in the Left Bar backed by a provider (Claude Code, Codex, etc.). This is what users see, create, and interact with. Each agent has its own workspace, tabs, and configuration.
- **Session** (or **provider session**) - An individual conversation context within a provider (e.g., Claude's `session_id`). Each AI tab within an agent can have its own provider session. In code, the `Session` interface represents an agent (historical naming).

Use "agent" in user-facing language. Reserve "session" for provider-level conversation contexts or when documenting the code interface.

### UI Components

- **Left Bar** - Left sidebar with agent list and groups (`SessionList.tsx`)
- **Right Bar** - Right sidebar with Files, History, Auto Run tabs (`RightPanel.tsx`)
- **Main Window** - Center workspace (`MainPanel.tsx`)
  - **AI Terminal** - Main window in AI mode (interacting with AI agents)
  - **Command Terminal** - Main window in terminal/shell mode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RunMaestro/Maestro](https://github.com/RunMaestro/Maestro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
