---
trigger: always_on
description: This repo (otto-code) is a fork with one mission: extend Otto into a **fully featured agentic coding assistant** - an IDE-grade environment with a rich feature set, familiar enough that you never feel constrained, that brings **frontier-model tooling to every provider equally, cloud and local alike**. The tooling a frontier harness gives its own model - browser-verified previews, artifacts, subagent visibility, context compaction, permission modes, MCP - should be just as available to a local mo
---

# AGENTS.md

## Why this fork exists

This repo (otto-code) is a fork with one mission: extend Otto into a **fully featured agentic coding assistant** - an IDE-grade environment with a rich feature set, familiar enough that you never feel constrained, that brings **frontier-model tooling to every provider equally, cloud and local alike**. The tooling a frontier harness gives its own model - browser-verified previews, artifacts, subagent visibility, context compaction, permission modes, MCP - should be just as available to a local model served from LM Studio as to a hosted frontier API. A capability isn't done when one provider has it; it's done when they all do.

The founding proof was the **Preview subsystem** - a rebuild of the Claude Code app's built-in `Claude_Preview` MCP server, shipped for all providers: agents start dev servers from a launch config, then verify browser-rendered changes (accessibility snapshots, DOM inspection, console/network capture, click/fill, viewport resize, screenshots), showing proof instead of asking the user to check manually. Read [docs/preview.md](docs/preview.md) before working on anything preview-related - it carries the design principles that must survive future changes (token economy, guardrail-bearing tool descriptions, daemon-enforced tab binding). The dev-server half lives in `packages/server/src/server/preview/`; the verification half is the daemon's browser-tools subsystem (`packages/server/src/server/browser-tools/`) executing against the Otto browser pane. Extend these; don't build a parallel browser stack.

The same leveling-up pattern has since shipped artifacts, the natively-tooled OpenAI-compatible provider (daemon-owned tool loop, MCP client, compaction, rewind), observed subagents for Claude, a provider-neutral git-hosting layer (GitHub + Bitbucket Cloud, see [docs/git-providers.md](docs/git-providers.md)), and agent profiles (named per-host templates with roles, spawnable by orchestrating agents, see [docs/agent-profiles.md](docs/agent-profiles.md)) - with remaining initiatives tracked as first-class project pages in Otto Knowledge. When adding a capability, design it provider-agnostic first and treat single-provider support as the proof, not the finish line.

## Repository map

`test-documents/` (repo root) holds hand-authored, self-contained fixtures for the file viewer - one per supported format, covering syntax highlighting and rendered previews. See [test-documents/README.md](test-documents/README.md). It is excluded from oxlint and oxfmt: the varied formatting is the point.

## Documentation

Four trees. Know which one you are in before you write anything down.

| Tree                                  | What it holds                                                                                | Tense               |
| ------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------- |
| **[`docs/`](docs/README.md)**         | The official software documentation - how Otto works. **This is the spec we build against.** | Present             |
| [`.otto/knowledge/projects/`](.otto/knowledge/projects/) | Project charters, delivery status, progress, and history; manage through Otto Knowledge      | Future and history  |
| [`.otto/knowledge/references/`](.otto/knowledge/references/) | External sources and their project-specific evaluation                                    | Present and history |
| [`projects/`](projects/README.md)     | Temporary read-only migration source for legacy charters; do not update                      | Legacy              |
| **Otto Knowledge** (`.otto/knowledge/`, or host-local when a project is configured that way) | The durable system-level architecture record (architecture, decisions, findings, projects, references) - retired from `archdocs/` | Present, durable, wide-angle |
| **This file**                         | Working rules for agents in this repo                                                        | Imperative          |

**The documentation index is [`docs/README.md`](docs/README.md).** Every chat receives the compact
active Knowledge catalog. At the start of non-trivial work, read the relevant confirmed Knowledge
pages and documentation before broad repository research.

**"The docs", "check the docs", or "check the X docs" always mean `docs/` - not the web.** Look there
before fetching anything online; it captures gotchas and conventions you cannot derive from the code
or from external sources.

`public-docs/` is the user-facing manual published to otto-code.me. Different audience, different
contract - it documents what Otto does, not how it is built. Do not put engineering notes there.

### Read these before touching the matching area


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Draek2077/otto-code](https://github.com/Draek2077/otto-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
