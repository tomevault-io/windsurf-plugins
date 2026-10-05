---
trigger: always_on
description: This file applies to the entire repository. It is the durable project brief for contributors and coding agents. Keep it aligned with the implementation and CI; do not turn it into a conversation log. Follow the user's current scope and instructions when they differ from these defaults.
---

# Working on Outpost

This file applies to the entire repository. It is the durable project brief for contributors and coding agents. Keep it aligned with the implementation and CI; do not turn it into a conversation log. Follow the user's current scope and instructions when they differ from these defaults.

## Purpose and philosophy

Outpost is a TypeScript library and CLI for running coding agents in sandboxes, managing their Git workspaces, preserving conversations and composing typed workflows.

- Prioritize reliable, directly usable behavior and an excellent developer experience.
- Make code understandable through names, small responsibilities and explicit contracts.
- Use domain-driven design and ports and adapters pragmatically. Introduce abstractions for real responsibilities and variations, not speculative flexibility.
- Keep agent protocols independent of sandbox backends. Claude Code, Codex, Antigravity (`agy`), GitHub Copilot CLI and Kimi Code have adapters; Kimi supports native capture, resume, fork and response repairs; Copilot supports capture, resume and repairs but rejects automated fork. Antigravity supports resume and repairs only in its existing sandbox, without portable capture or automated fork. Additional agents belong behind the existing ports.
- Harness model requests use the stable `ModelProvider` contract and `createOpenAIModelProvider()` / `createAnthropicModelProvider()` in `src/adapters/models/`, independently of sandbox providers and CLI agent adapters. Providers exchange bounded messages with tool calls and replayable reasoning, with optional streaming. Built-in subagents use `defineHarnessSubagent()` with a borrowed sandbox, intersected declarative permissions, separate histories and cumulative ancestor token budgets.
- Preserve existing features and public contracts during refactoring. Architecture changes must not silently change execution behavior.
- Prefer explicit ownership, predictable failure modes and recoverable state over hidden automation.
- Distinguish released behavior, implemented but unreleased additions, and opt-in research prototypes. Keep remaining work and live-validation prerequisites in the roadmap; do not imply publication from a local implementation.

## Start here

Read `README.md`, `package.json`, `SECURITY.md` and `docs/src/content/docs/guide/integration-ports.md`. Inspect the relevant source, tests and workflows before editing. Check Git status and preserve unrelated work.

Use the repository as the source of truth for versions, supported options and commands. Do not rely on test counts, coverage percentages or publication status remembered from a previous chat.

- Runtime: Node.js 24+, TypeScript, ESM. Bun (version pinned by `packageManager`) installs dependencies and runs scripts, with committed `bun.lock` files; tests and scripts still execute on Node.js. `images/agents/` keeps its npm lockfile because the agent image installs CLIs with npm.
- Public package: `@elie-laloum/outpost`.
- Public facade: `src/index.ts` and the provider subpaths declared in `package.json`.
- Canonical repository: <https://gitlab.elielaloum.com/elielaloum/outpost>.
- GitHub mirror and CI: <https://github.com/elie-laloum/outpost>.
- Documentation: <https://elie-laloum.github.io/outpost/>.

## Architecture and responsibilities

| Location               | Responsibility                                                                               |
| ---------------------- | -------------------------------------------------------------------------------------------- |
| `src/domain/`          | Contracts, validation, prompts, responses, usage, task graphs and workflow rules.            |
| `src/application/`     | Use cases, resource ownership, dispatch, lifecycle orchestration and remote synchronization. |
| `src/adapters/agents/` | One folder per agent: request building, event decoding, descriptor. Plus the catalog.        |
| `src/providers/`       | Sandbox allocation, command execution, file transfer and disposal.                           |
| `src/infrastructure/`  | Processes, binary streams, Git, files, native conversation storage and logging.              |
| `src/cli/`             | Argument handling, project scaffolding and image commands.                                   |

`scripts/check-architecture.mjs` enforces these internal dependency directions:

- Domain depends only on domain modules.
- Infrastructure may depend on domain and infrastructure.
- Adapters may depend on domain, infrastructure and adapters.
- Providers may depend on domain, infrastructure, adapters and providers.
- Application may depend on every non-CLI layer.
- CLI composes the other layers.

These are allowed boundaries, not a reason to add unnecessary dependencies. Keep vendor SDKs and protocol details out of the domain. Optional cloud SDKs must remain optional and load through their provider entry points.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elie-laloum/outpost](https://github.com/elie-laloum/outpost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
