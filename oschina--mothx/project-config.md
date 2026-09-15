---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Read this file before exploring or editing code. Keep changes focused, preserve existing behavior and APIs, and validate the smallest relevant scope.
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Read this file before exploring or editing code. Keep changes focused, preserve existing behavior and APIs, and validate the smallest relevant scope.

## Project snapshot

- **Primary language:** Go 1.27 (`go.mod`), with a Cobra CLI and Bubble Tea/Lipgloss TUI.
- **Frontend:** Svelte 5 + Vite in `ui/`; the built UI is embedded into the Go binary.
- **Desktop:** Electron + TypeScript in `desktop/`; this is a first-class, **pure ACP** client. It packages a source-built `mothx acp` runtime and has its own React 19 + shadcn/ui + Tailwind CSS renderer (Vite-built classic scripts for `file://`); it neither starts `mothx serve` nor embeds/reuses the Svelte Web UI.
- **Packaging:** npm installer packages under `npm/` and a Python installer package under `pypi/`.
- **Purpose:** MothX (`mothx`) is a terminal AI coding assistant with provider adapters, streaming agent execution, tools, sessions, sandboxing, skills, workflows, serve/API mode, messaging channels, and SDK support.

## Important directories

- `cmd/mothx/` — Cobra CLI entry point and subcommands (`serve`, `stats`, `a2a`, `doctor`, etc.).
- `agent/` — public Go SDK types and interfaces. Treat changes here as public API changes.
- `bootstrap/` — blank-import wiring that connects public SDK types to internal implementations.
- `example/` — public SDK examples.
- `internal/agent/` — core agent loop, events, context handling, tool execution, sub-agents, and system prompts.
- `internal/agentruntime/` — the authoritative front-end-neutral runtime layer: shared `SessionRuntime`/`Builder` resource assembly, `ResolveSource`/`ResolvePolicy` mode wiring, `ExecutionRuntime` durable run lifecycle, `DecisionService`/`DecisionRecord` replay, MCP lifecycle, and coordinated shutdown for TUI, CLI, WebUI/API, channels, and ACP. See `docs/proposal/agent-core-runtime-unification-proposal.md`.
- `internal/expert/` — discovery and validation of built-in, global, and project Expert/Expert Team bundles. A session's resolved identity and team capability remain Runtime-owned in `internal/agentruntime`.
- `internal/provider/` — provider abstraction and implementations; `anthropic/`, `google/`, and `openai/` contain full providers, while `vendor_*.go` contains vendor detection/defaults.
- `internal/provider/factory/` — shared provider/model construction. Use this from CLI, ACP, serve, and other runtimes.
- `internal/tools/` — built-in tools and tool registration.
- `internal/tui/` — Bubble Tea terminal UI.
- `internal/serve/` — unified server runtime: OpenAI-compatible API, Web UI, channels, hooks, cron, memory, and settings APIs.
- `internal/serve/openaiapi/` — HTTP API handlers, slash commands, and tool-output formatting.
- `internal/architecture/` — static architecture guard tests that forbid production code from reintroducing direct `agent.New`/`agent.NewWithLoopConfig`, bypassing canonical Run persistence, or bypassing the DB-to-DAO boundary. Keep the allowlist minimal and documented; run `go test ./internal/architecture` after touching production call sites.
- `internal/db/` — process-wide Bun connection ownership, SQLite configuration, migration startup, and database shutdown.
- `internal/dao/` — the only production owner of Bun query construction, SQL statements, table persistence, and row mapping.
- `internal/session/` — Session domain APIs and replay logic; it uses `internal/db` for managed handles and `internal/dao` for every database operation.
- `internal/config/` — `settings.json` schema, defaults, and configuration persistence.
- `internal/contextfiles/`, `internal/skills/`, `internal/workflow/` — project context discovery, reusable skills, and workflow execution. Built-in skill guidance lives under `internal/skills/builtin/`; project/global skills remain user-owned overrides.
- `internal/sandbox/`, `internal/mcp/`, `internal/acp/`, `internal/a2a/` — sandboxing and protocol integrations. `internal/mcp/server.go` owns reusable stdio MCP protocol serving; domain packages provide handlers rather than implementing their own JSON-RPC loop.
- `internal/stats/` — usage statistics dashboard and queries.
- `ui/src/` — Svelte application; `App.svelte` routes views, `lib/stores.js` owns shared stores, `lib/preferences.js` owns `zh`/`en` translations, and `style.css` contains global styles.
- `desktop/` — the primary desktop product: Electron main process (`main/`), restricted preload bridge (`preload/`), separate React/shadcn renderer (`renderer/`: `core/` DOM-free state + ACP actions bridged to React via `useSyncExternalStore`, `components/ui/` shadcn primitives, `views/` screens and settings panels), tests, runtime-vendoring/build scripts, and packaging configuration. `main/acp-client.ts` is the only ACP client; renderer code only uses `window.mothx` through the preload bridge.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oschina/mothx](https://github.com/oschina/mothx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
