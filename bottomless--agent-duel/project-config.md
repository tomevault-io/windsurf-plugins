---
trigger: always_on
description: This repository is Agent Duel, a fork of Paseo built
---

# Agent Duel agent guide

This repository is Agent Duel, a fork of Paseo built
around blinded coding-agent battles. Preserve Paseo's local daemon and client
architecture. Treat Electron as the product and the browser build as a local
development and QA harness only. An unspecified request to run or test the app
means the Agent Duel desktop stack, not the upstream Paseo daemon by itself.

A battle runs two blinded contestants against one prompt in isolated worktrees, and the winner you
vote for becomes the chat's history. Read [docs/arena.md](docs/arena.md) before working on any of
it — the engine lives outside the npm workspace and none of the inherited Paseo docs describe it.

**Supported agents:** Claude Code, Codex, GitHub Copilot, OpenCode, and Pi.

## Repository map

This is an npm workspace monorepo. `arena-backend` is not part of it — it is a vendored OpenCode
fork with its own Bun install.

- The hosted control plane lives in a separate private repository; this repository reaches it only over HTTP
- `arena-backend/packages/arena-service` — Battle routes shared with the control plane: assignments, OpenRouter proxy, comparison
- `arena-backend/packages/opencode` — Local battle engine: blinding, worktrees, voting, canonical git and transcripts; compiled into desktop
- `packages/app` — Electron renderer and browser QA harness (Expo); battle UI in `packages/app/src/arena`
- `packages/server` — Daemon: agent lifecycle, WebSocket API, MCP server, Arena bridge
- `packages/desktop` — Electron desktop wrapper, the primary target
- `packages/protocol` / `packages/client` — Wire schemas and the client SDK
- `packages/cli` — Docker-style CLI (`paseo run/ls/logs/wait`)
- `packages/relay` — E2E encrypted relay for remote access

## Docs

`docs/` is the source of truth for system-level and process-level knowledge. **"The docs", "check the docs", or "check the X docs" always mean this directory — not the web.** Look here before fetching anything online; the docs capture gotchas and conventions you cannot derive from the code or external sources.

These are the docs this product is built on. Read the relevant one before non-trivial work.

| Doc                                                  | What's in it                                                                                                                    |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| [docs/arena.md](docs/arena.md)                       | **The battle.** What it is for, settled product decisions, turn lifecycle, blinding, voting, states                             |
| [docs/accounts.md](docs/accounts.md)                 | **Sign-in.** Magic link, Google, GitHub; where accounts live, how the session travels, the ordering traps                       |
| [docs/architecture.md](docs/architecture.md)         | System design, package layering, WebSocket protocol, agent lifecycle, data flow                                                 |
| [docs/glossary.md](docs/glossary.md)                 | Authoritative terminology — UI label wins, no synonyms                                                                          |
| [docs/coding-standards.md](docs/coding-standards.md) | Type hygiene, error handling, state design, React patterns, file organization                                                   |
| [docs/design.md](docs/design.md)                     | Design system — tokens, buttons, hierarchy, density, alignment rails, states, what's forbidden                                  |
| [docs/side-panel.md](docs/side-panel.md)             | The workspace's main pane + side panel — which tabs go where, the two-pane layout shape, open state, what replaced the Explorer |
| [docs/forms.md](docs/forms.md)                       | Form architecture — non-React form model, form kit, load-state gating; the agent profile form is the golden example             |
| [docs/hover.md](docs/hover.md)                       | Hover — the canonical pattern (plain View + onPointerEnter/Leave, separate inner Pressable) and the three ways agents break it  |
| [docs/unistyles.md](docs/unistyles.md)               | Unistyles gotchas — `useUnistyles()` is forbidden, alternatives in order                                                        |
| [docs/floating-panels.md](docs/floating-panels.md)   | Anchored popovers — Portal/Modal escape for Android, lifecycle gates, keyboard-shared-value, status-bar offset, the flash       |
| [docs/menus.md](docs/menus.md)                       | The menu engine — popover vs sheet, submenu pages, hover intent, when a decision earns a submenu                                |
| [docs/expo-router.md](docs/expo-router.md)           | Expo Router route ownership, startup restore, and native blank-screen gotchas                                                   |
| [docs/development.md](docs/development.md)           | Dev server, build sync gotchas, CLI reference, agent state, Playwright MCP                                                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bottomless/agent-duel](https://github.com/bottomless/agent-duel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
