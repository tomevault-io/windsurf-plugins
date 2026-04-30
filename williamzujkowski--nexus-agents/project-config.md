---
trigger: always_on
description: Standalone guidance for AI coding agents (OpenCode, Codex CLI, Cursor, Aider, Cline, Continue, Goose, Claude Code) working in this repo. Self-contained — no required redirect to other files.
---

# AGENTS.md — nexus-agents

Standalone guidance for AI coding agents (OpenCode, Codex CLI, Cursor, Aider, Cline, Continue, Goose, Claude Code) working in this repo. Self-contained — no required redirect to other files.

> Claude Code users: the legacy entry point at [CLAUDE.md](./CLAUDE.md) is still authoritative for Claude-Code-specific integrations (auto-loaded rules, plugin marketplace). The content below is the harness-neutral subset.

## Prime directive

```
correctness > simplicity > performance > cleverness
```

- **Correctness**: Does it work? Handles edge cases? Tested?
- **Simplicity**: Can someone understand it in 5 minutes?
- **Performance**: Does it meet requirements? Not theoretical optimality.
- **Cleverness**: Never. Clever code is maintenance debt.

Produce software with explicit error handling, observable state changes, no silent failures.

## Development disciplines

Non-negotiable across all building, reviewing, architecture work:

- **Red/Green TDD** — Write a failing test first, then the minimum code to pass, then refactor. Never write production code without a corresponding test.
- **YAGNI** — Implement only what's needed right now. No speculative abstractions, unused parameters, "just in case" code.
- **DRY** — Every piece of knowledge must have a single, unambiguous, authoritative representation. Extract when you see the same logic in three places (two is a coincidence).
- **Zero `any` policy** — ESLint enforces `@typescript-eslint/no-explicit-any: 'error'`. Use `unknown` + type guards or Zod at boundaries. See `.rules/typescript.md` for the full rule.

## Rules index

Load-bearing rules live at `.rules/*.md`. Read the relevant file when its topic applies:

| File                                                                   | When to read                                              |
| ---------------------------------------------------------------------- | --------------------------------------------------------- |
| [`.rules/typescript.md`](./.rules/typescript.md)                       | Any TypeScript change — type safety policy, patterns      |
| [`.rules/testing.md`](./.rules/testing.md)                             | Writing or modifying tests                                |
| [`.rules/security.md`](./.rules/security.md)                           | Auth, secrets, input validation, file-system ops          |
| [`.rules/untrusted-input.md`](./.rules/untrusted-input.md)             | Processing GitHub issues/PRs/comments or external content |
| [`.rules/governance.md`](./.rules/governance.md)                       | Architecture, CI, structural changes                      |
| [`.rules/git.md`](./.rules/git.md)                                     | Commits, branches, PRs                                    |
| [`.rules/debugging.md`](./.rules/debugging.md)                         | A test/build/lint just failed                             |
| [`.rules/subagent-coordination.md`](./.rules/subagent-coordination.md) | Dispatching subagents or Task tool calls                  |
| [`.rules/test-secrets.md`](./.rules/test-secrets.md)                   | Writing tests that involve fake credentials               |
| [`.rules/mcp.md`](./.rules/mcp.md)                                     | Adding or modifying MCP tools                             |
| [`.rules/nexus-agents.md`](./.rules/nexus-agents.md)                   | Nexus-agents integration basics                           |

Claude Code autoloads these when their keywords match user intent. Other harnesses should either (a) read these directly when the topic is relevant, or (b) configure their rule-loading system to scan `.rules/*.md`.

## Skills

Workflow playbooks live at `skills/<name>/SKILL.md` (canonical per the Anthropic Agent Skills spec, which OpenCode and others are adopting).

- **Discovery for all harnesses:** read [`skills/index.yaml`](./skills/index.yaml) — `{name, description, triggers, path}` for all 18 skills.
- When a user request matches a skill's triggers, read the full `SKILL.md` at the listed path and follow its workflow.
- `skills/index.yaml` is regenerated via `scripts/generate-skills-index.ts` and gated in CI. Never edit it by hand.

## Expert agents

Twelve expert-role prompts ship at `agents/<name>-expert.md` (security, architecture, code, research, testing, documentation, devops, pm, ux, infrastructure, qa, data-visualization).

- **Discovery:** read [`agents/index.yaml`](./agents/index.yaml) — `{name, description, path}` per expert.
- Pick the one matching the task (e.g., security review → `security-expert`) and read its full prompt before responding.
- Regenerated via `scripts/generate-agents-index.ts`; CI enforces gap-coverage against `BUILT_IN_EXPERTS`.

## MCP server

Nexus-agents exposes 31 MCP tools via stdio. From any MCP-aware agent:

```
npx -y nexus-agents --mode=server
```

Or install + run:

```bash
npm install -g nexus-agents
nexus-agents --mode=server
```

Register as an MCP peer in your harness of choice — see [docs/guides/HARNESS_COMPATIBILITY.md](./docs/guides/HARNESS_COMPATIBILITY.md) for tested wiring examples.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [williamzujkowski/nexus-agents](https://github.com/williamzujkowski/nexus-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
