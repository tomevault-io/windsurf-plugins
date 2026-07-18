---
trigger: always_on
description: pier is a TypeScript + Bun coding-agent CLI. It runs an interactive REPL (or headless
---

# pier — AGENTS.md

pier is a TypeScript + Bun coding-agent CLI. It runs an interactive REPL (or headless
`exec`/`serve`), drives an agentic tool loop against models served through the Pier bridge
(with BYOK support for Sarvam AI and OpenRouter), and ships as a single compiled binary.

## Quick reference

| Command | What it does |
|---------|-------------|
| `bun test` | Run all tests (bun:test) |
| `bun run dev` | Run without compiling (`bun src/entry.ts`) |
| `bun run build` | Compile → `dist/pier` |
| `bun run typecheck` | `tsc --noEmit` |
| `bun run lint` | `biome check src scripts` |
| `bun run format` | `biome format --write src scripts test` |
| `bun run verify` | typecheck + lint + format:check + test |

Run a single test file first: `bun test test/<file>.test.ts`, then the full suite.

## Project structure

```
src/
  entry.ts              — bin entry shim (loads MACRO global, boots SRT, calls runCli)
  cli.ts                — Commander CLI multitool (repl, exec, serve, login, keys, …)
  ink.ts                — thin facade over the ink renderer + design-system components
  _macros/              — build-time constants (BUILD, feature flags, MACRO global)
  protocol/             — Responses API wire types (items.ts), model catalog (models.ts),
                          approval/sandbox types, bundled models_catalog.json
  bridge/               — model client: SSE frame parser, event validator, POST /v1/responses,
                          GET /v1/models, tips, subscription
  auth/                 — device pairing (OAuth device-code flow) + token storage
  engine/               — the turn loop: callModel, translate-in/out, turn.ts (agentic loop),
                          session.ts (mutable state), controller.ts (headless session driver),
                          exec.ts (headless path), compact/microcompact, retry, fallback,
                          checkpoint, planFile, ultracode, backgroundTasks, monitors
  tools/native/         — native tool handlers (Read, Write, Edit, Grep, Bash, Task, …)
                          + registry.ts (filters tools by permission mode)
  safety/               — permission engine, sandbox adapter, trust model, auto-classify
  hooks/                — lifecycle hooks (PreToolUse, PostToolUse, SessionStart, Stop, …)
  agents/               — subagent loading + execution (built-in + disk-defined)
  skills/               — SKILL.md-based skill bundles
  mcp/                  — MCP client (stdio + streamable-http)
  commands/             — slash commands + custom commands from $PIER_HOME/commands/*.md
  context/              — project memory (CLAUDE.md / AGENTS.md discovery + @import)
  providers/            — BYOK provider registry (Sarvam AI, OpenRouter), chat completions
                          client, credential management, model resolution
  server/               — `pier serve` headless session server (stdio + register-out WS)
  runner/               — runner adapter for the bridge's runner pool
  threads/              — thread persistence (local JSON store + bridge sync)
  tui/                  — REPL: composer, transcript cells, dialogs, onboarding, model picker
  terminal/             — ANSI rendering, hyperlinks, clipboard, titled panes, string width
  components/           — design-system (ThemedBox, ThemedText, ThemeProvider, color)
  config/               — build channel, PIER_HOME resolution, internal-access gate
  utils/                — config, fs, log, sanitize, schedule, theme, env, debug, …
  types/                — message.ts (internal message union the whole engine/UI speaks)
packages/
  protocol/             — @pier/serve-protocol: Zod schemas for the serve NDJSON protocol
test/                   — ~100 test files, one per module/feature (bun:test, no describe/it)
scripts/                — build, release, install, model sync, vendor-rg
vendor/rg/              — bundled ripgrep binaries (darwin, linux, windows, arm64/x64)
```

## Key architecture

- **Entry flow**: `entry.ts` → `cli.ts` → `launchRepl()` (TUI) or `runExec()` (headless) or
  `runServe()` (embedding server). The REPL creates a `Session` + `SessionController` and
  renders a `Repl` component through ink.
- **Turn loop** (`engine/turn.ts`): streams a model call via `callModel`, collects tool_use
  blocks, gates each through the approval policy (permission mode + safety layer), runs the
  tool, appends tool_result, and loops until `stop_reason !== 'tool_use'`.
- **Session state** (`engine/session.ts`): a mutable `Session` object threads through the
  whole turn loop — cwd, model, instructions, history, permission mode, sandbox policy,
  token usage, checkpoints, todos, and the tool allowlist.
- **Permission modes**: `default` (prompt on writes/exec), `plan` (read-only + ExitPlanMode),
  `acceptEdits` (auto-approve edits), `auto` (classifier decides, fails safe to prompt),
  `bypassPermissions` (everything auto-approved). Cycled with Shift-Tab in the REPL.
- **OS sandboxing**: Bash runs inside a real OS sandbox (seatbelt on macOS, bubblewrap on
  Linux) sized to the active permission mode. The sandbox blocks outbound network by default;
  `needs_network: true` requests it per command.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alphabench/pier](https://github.com/alphabench/pier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
