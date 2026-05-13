---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

This is a **dependency-free Node.js project** (no `node_modules`, no bundler). Everything runs against the Node 18+ standard library.

```bash
npm run ci                            # full CI gate: npm test + check:docs
npm test                              # local code gate: check:node + check:shell + test:e2e
npm run check:node                    # node --check on every script (syntax only)
npm run check:docs                    # docs/i18n/secret/BOM gate (scripts/check-docs.mjs)
npm run check:shell                   # bash -n on installer + service scripts
npm run test:e2e                      # node --test over tests/e2e/*.e2e.mjs (excludes real-clients)
npm run test:e2e:real                 # opt-in: requires LEGAX_REAL_CLIENTS=1 and real codex/Codex/gemini CLIs on PATH
npm run test:real:messages            # real-local-message smoke test
```

Run a single E2E file: `node --test tests/e2e/<name>.e2e.mjs`. Run a single test inside it: `node --test --test-name-pattern '<substring>' tests/e2e/<name>.e2e.mjs`.

`npm run ci` is the contract enforced by CI; if you add a new script or test file, also append it to the corresponding `check:node` / `test:e2e` list in `package.json` — the lists are explicit, not glob-driven.

Dry-run the supervisor without spawning real CLIs: `node scripts/legax-daemon.mjs --dry-run`.

## Contribution and commits

Follow `.github/CONTRIBUTING.md` when preparing changes for commit:

- Treat `npm run ci` as the full merge gate (`npm test` plus `check:docs`). For targeted bugfix work, run the narrow regression tests first, then the relevant broader gate, and state any full-gate failures explicitly instead of implying a clean CI.
- Documentation-only commits may use `npm run check:docs` locally, but code commits should at least run the affected E2E file(s), `npm run check:node`, and any daemon/transport integration tests touched by the change.
- Keep documentation pairs in the same commit: English `.md` with `.zh-CN.md`, and `config.example.yaml` with `config.example.zh-CN.yaml`.
- Before staging, inspect `git status --short` and the full diff. Do not stage local runtime artifacts, large binaries, `data/` residue, machine-specific config, lock files, or tracked secrets.
- Commit subjects should be short imperative sentences. Conventional Commits are not required; a topic prefix is preferred when it helps scanning, for example `telegram: refresh active pinned context`, `daemon: route opencode launches`, or `docs: clarify relay pairing`.
- Use a commit body only when the *why* is not obvious from the subject and diff.

## Architecture

The project is a local-first relay layer that bridges coding-agent CLIs (Codex, Claude Code, Gemini CLI, OpenCode) to a phone via self-hosted relay, Telegram, or webhook. Read `docs/ARCHITECTURE.md` first — the design uses **three planes** that must not be conflated when changing code:

- **Control plane (CLI adapters)** — owns process lifecycle, session selection/continuation, and structured-output parsing. One adapter per agent under `scripts/`: `codex-app-server-link.mjs` (JSON-RPC over WebSocket app-server), `claude-code-link.mjs` (`claude -p` stream-json), `gemini-cli-link.mjs` (`gemini` stream-json), `opencode-link.mjs` (OpenCode HTTP server). Each adapter is a long-lived process; the daemon supervises them.
- **Capability plane (MCP)** — `scripts/mcp-server.mjs` is a generic stdio MCP server exposing `legax_send/poll/request_permission/status`. `scripts/claude-permission-mcp-server.mjs` is a Claude-specific permission-prompt MCP that mirrors permission asks to the phone and returns the decision through Claude's permission hook. MCP is a **capability layer, not a lifecycle manager** — never use it to start/stop processes.
- **Communication plane (transports)** — `scripts/lib/outbound-transports.mjs` (relay POST, Telegram sendMessage, generic webhook) and `scripts/lib/inbound-transports.mjs` (Telegram parsing/routing helpers). The daemon owns relay `/api/messages` polling and Telegram `getUpdates` while running, writes messages into per-agent inbox queues, and adapters launched by the daemon only drain their inbox. Standalone adapters use a single-poller fallback.

The **unified daemon** (`scripts/legax-daemon.mjs`) is what users normally run. It reads one `config.yaml`, owns remote inbound routing, supervises all enabled adapters, restarts crashed ones with bounded backoff, writes per-adapter MCP config (`mcpAutoConfigure`) before launch, and handles **on-demand launches** for `autoStart: false` adapters via launch requests in runtime state. If only an individual adapter is running, it cannot start siblings.

### Cross-process coordination via runtime state


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhanex/legax](https://github.com/zhanex/legax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
