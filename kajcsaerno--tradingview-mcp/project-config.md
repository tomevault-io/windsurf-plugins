---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

78 MCP tools (and matching `tv` CLI commands) for reading and controlling a live TradingView Desktop chart via Chrome DevTools Protocol on `localhost:9222`. Two consumers, one core: an MCP server (stdio) and a pipe-friendly CLI.

## Development Commands

```bash
npm install                        # zero-config, only @modelcontextprotocol/sdk + chrome-remote-interface
npm start                          # run MCP server (stdio)
npm run tv -- <command>            # run CLI (or `node src/cli/index.js`)

npm test                           # e2e + pine_analyze (e2e REQUIRES TradingView running on :9222)
npm run test:unit                  # pine_analyze + cli — no TradingView needed
npm run test:cli                   # CLI router tests
npm run test:e2e                   # full e2e (needs live TradingView)
npm run test:verbose               # spec reporter
node --test tests/sanitization.test.js   # CDP injection-prevention tests (pure unit, no TV)
node --test tests/replay.test.js         # replay logic unit tests

# Run a single test by name filter
node --test --test-name-pattern="setSymbol" tests/sanitization.test.js
```

There is no lint/format step configured. Tests use the built-in `node:test` runner — no Jest, Mocha, or Vitest.

## Code Architecture

The codebase is a strict three-layer fan-out from a single core. **All TradingView interaction lives in `src/core/`; everything else is a thin adapter.**

```
                            ┌──────────────────────┐
   MCP client (Claude) ───► │  src/tools/*.js      │ ──┐
                            │  (Zod schemas +      │   │
                            │   server.tool calls) │   │
                            └──────────────────────┘   │
                                                       ▼
                            ┌──────────────────────┐  src/core/*.js
   CLI (`tv` command) ────► │  src/cli/commands/   │ ──► (chart, data, pine,
                            │  (parseArgs router)  │     replay, drawing, …)
                            └──────────────────────┘     │
                                                         ▼
                                                src/connection.js
                                                (CDP client, evaluate(),
                                                 safeString, requireFinite)
                                                         │
                                                         ▼
                                              CDP :9222 → TradingView
```

### Layers

- **`src/connection.js`** — singleton CDP client with auto-reconnect (exponential backoff up to 5 tries). Picks the Electron target whose URL matches `tradingview.com/chart`. Exposes `evaluate(expr)` / `evaluateAsync(expr)` which `Runtime.evaluate` JS strings inside the TradingView page. Also exports the **two security primitives every core function uses**:
  - `safeString(str)` — `JSON.stringify`-based escape for any user value interpolated into evaluated JS. Mandatory for symbols, indicator names, dates, drawing text, etc.
  - `requireFinite(n, name)` — validator for numeric inputs before they reach TV APIs that persist to cloud state.
  - `KNOWN_PATHS` — verified deep paths into the TradingView Electron app (`window.TradingViewApi._activeChartWidgetWV.value()`, `_replayApi`, `_alertService`, pine-facade REST, etc.). These are undocumented and **can break on any TradingView update** — pin the desktop version if stability matters.

- **`src/wait.js`** — `waitForChartReady()` polls DOM (loader spinner, bar count stability, symbol-match) before returning control. Use this after any chart mutation; do not rely on `setTimeout`.

- **`src/core/*.js`** — pure logic. Each module (chart, data, pine, replay, drawing, alerts, batch, indicators, pane, tab, ui, watchlist, capture, health, stream) exports functions that build a JS expression string, call `evaluate()`, and return `{ success, … }`. **Every function takes an optional `_deps` parameter** that defaults to the real `evaluate`/`evaluateAsync`/`waitForChartReady`. This is the DI hook used by `tests/sanitization.test.js` and `tests/replay.test.js` to assert against the actual JS strings being sent — that is how injection regressions are caught without a live chart.

- **`src/tools/*.js`** — MCP adapters. Each `register*Tools(server)` registers one `server.tool(name, description, zodSchema, handler)` per core function. Handlers are uniformly `try { return jsonResult(await core.fn(args)); } catch (err) { return jsonResult({ success: false, error: err.message }, true); }`. The `_format.js` helper wraps payloads as MCP text content.

- **`src/cli/`** — `index.js` imports every command module; `router.js` is a zero-dep wrapper over `node:util.parseArgs` with subcommand support; `commands/*.js` map CLI flags to the same `src/core/*` functions. CLI prints JSON to stdout, errors to stderr, exit codes: 0 ok / 1 error / 2 connection failure. `tv stream` polls and diffs to emit JSONL.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KajcsaErno/tradingview-mcp](https://github.com/KajcsaErno/tradingview-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
