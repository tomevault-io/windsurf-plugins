---
trigger: always_on
description: MCP server that wraps AI CLI tools (Claude Code, Gemini CLI, Codex CLI) as MCP tools for any MCP client.
---

# mcp-agents

MCP server that wraps AI CLI tools (Claude Code, Gemini CLI, Codex CLI) as MCP tools for any MCP client.

## Architecture

The primary ESM server lives in `server.js`; there is no build step or
transpilation. `CLI_BACKENDS` defines the blocking Claude/Gemini commands,
while browser and the default Codex provider have dedicated proxy/adapter
runtimes. The deprecated native MCP implementation is sealed in
`codex-legacy.js` and loaded only for `--provider codex-legacy`, making its
eventual removal one explicit provider/module boundary. Version is read from
`package.json` at runtime via `readFileSync` (not import assertions — the syntax
shifted across Node versions and the runtime read sidesteps that entirely).

## Commands

```sh
# Run server
node server.js --provider claude   # or: gemini, codex (default), codex-legacy

# Tests (fast, no real CLI calls)
SKIP_INTEGRATION=1 ./test.sh

# Tests (full, calls real CLIs — requires claude/gemini/codex installed)
./test.sh

# Verify CLI flags
node server.js --help
node server.js --version
```

## Local Install

```sh
npm install && npm link
```

`.tool-versions` pins the checkout's Node for mise. `npm link` puts `mcp-agents`
in the *active* Node's global prefix, so switching Node versions orphans the
link — re-run `npm link` if `mcp-agents` disappears from `$PATH`.

## Critical: stdout is MCP-only

NEVER write to stdout in server mode — it's the MCP JSON-RPC transport. Use `logErr()` (writes to stderr) for all logging. `console.log` is only safe in `printHelp()` / `parseArgs()` which call `process.exit()` before the server starts.

## Gotchas

- `package.json` must stay in the `files` array — the server reads it at runtime for `VERSION`
- Child process stdin must be closed immediately (`child.stdin?.end()`) or the CLI hangs waiting for EOF
- The `keepAlive` interval prevents premature exit when stdin EOF arrives before async handlers complete
- `engines` requires `>=26` — raised in 0.28.0 so the browser provider's downstream floor is always satisfied. Older providers ran on 18; do not reintroduce a lower floor without checking `chrome-devtools-mcp`
- `codex` and `codex-legacy` are intentionally separate providers with no
  automatic fallback. `codex` owns the App Server migration and all new Codex
  functionality. `codex-legacy.js` must preserve the complete 0.28 native
  `codex mcp-server` behavior while that upstream command remains supported;
  do not partially reimplement it, route it through App Server, or add App
  Server-only tools/state semantics. Removing the legacy provider later must
  be an explicit breaking change
- Keep `codex-legacy.js` self-contained except for its narrow exported runner.
  Deliberate helper duplication is the deletion boundary, not an invitation to
  couple legacy framing/auth/job behavior back into the App adapter. Its
  private homes remain under `${STARTUP_CWD}/tmp/codex-homes/` for exact
  compatibility; do not silently move them into App durable state
- The `codex` provider is a wrapper-owned MCP server over the documented, version-gated stdio JSONL surface of `codex app-server`; it is NOT a native MCP pass-through. Outer `initialize`, `tools/list`, `ping`, validation, progress, jobs, and local state tools must stay independent of child availability. Never forward raw App Server frames to MCP stdout and never initialize with `experimentalApi`
- Codex tools expose closed wrapper-owned schemas. `codex` requires `prompt`, an **absolute** `cwd`, and `sandbox`, with optional curated model/effort, `allow_subagents`, and native `goal`; `codex-reply` requires `prompt` plus nonblank `threadId`. App Server config/filesystem methods, raw instructions, providers, reply execution settings, and per-call approval policy remain hidden. Validation errors are redacted MCP `-32602` responses and never reach App Server
- App Server children are lazy and generation-scoped. Every pending native request, interaction, and turn carries its generation; late frames from a dead generation are ignored. A child exit rejects already-dispatched turns as `codex_outcome_unknown`, records that state in the sidecar, and NEVER replays them. Only a later safe operation may spawn the next generation
- Completion authority is the stable `turn/completed` notification plus completed `agentMessage` / `exitedReviewMode` items. Deltas are presentation-only. Preserve exactly one MCP result, retain the durable `threadId`, and treat `interrupted` / `failed` as errors even if partial agent text exists
- Durable Codex state is external and project-scoped at `${XDG_STATE_HOME:-$HOME/.local/state}/mcp-agents/codex/projects/<sha256(canonical STARTUP_CWD)>/v1` unless overridden. The allowlist is `sessions/`, `archived_sessions/`, native `thread-writer-locks/`, the native goal store, wrapper `leases/`, retention metadata, and content-free `bridges/` sidecars. Reject roots inside the served workspace, set umask `0077`, and keep directories `0700` / files `0600`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomaswitt/mcp-agents](https://github.com/thomaswitt/mcp-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
