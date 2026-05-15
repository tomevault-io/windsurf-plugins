---
trigger: always_on
description: An AI trading desk with 45 hedge fund agent personas (including 22 named personas like Arthur Hayes, Jim Simons, George Soros, Jesse Livermore, Warren Buffett, and Peter Lynch) for Claude Code. 150 MCP tools across 3 built-in connectors. 28 shared analysis libraries. Trade on any exchange — Cube, OKX, Kraken, Binance, Coinbase, and 100+ more via CCXT.
---

# AI Fund

An AI trading desk with 45 hedge fund agent personas (including 22 named personas like Arthur Hayes, Jim Simons, George Soros, Jesse Livermore, Warren Buffett, and Peter Lynch) for Claude Code. 150 MCP tools across 3 built-in connectors. 28 shared analysis libraries. Trade on any exchange — Cube, OKX, Kraken, Binance, Coinbase, and 100+ more via CCXT.

## Project Structure

```
ai-fund/
├── skills/              # 44 agent personas (SKILL.md each) + _template/
├── connectors/cube/     # Built-in Cube Exchange MCP server
│   └── mcp-server/
├── connectors/alpaca/   # Built-in Alpaca MCP server (stocks, ETFs, crypto)
│   └── mcp-server/
│       ├── src/cli/         # device-login, login, logout, status
│       ├── src/client/      # iridium (REST), osmium (WS), auth, signing, credential-store
│       ├── src/tools/       # market-data, orders, account, defi, risk
│       ├── src/resources/   # markets, portfolio
│       └── tests/           # vitest test suites
├── connectors/ccxt/     # Built-in CCXT MCP server (Coinbase, Binance, 100+ exchanges)
│   └── mcp-server/
│       ├── src/cli/         # status
│       ├── src/client/      # exchange wrapper, rate limiter, latency tracker, trade journal
│       ├── src/tools/       # market-data, orders, account, strategy, execution, datastore
│       └── tests/           # vitest test suites (190+ tests)
├── lib/                 # Shared TS: indicators, math, format
├── bin/desk-state       # CLI for .desk/ state management
├── scripts/install.js   # npx ai-fund install|list
├── .claude/commands/    # Slash commands (hire, fire, desk, review, setup, backtest)
├── docs/                # Architecture diagram, auth brief
├── examples/            # Preset desk configurations (JSON)
└── .desk/               # Runtime state (gitignored, per-user)
```

## Architecture

- **Skills** (`skills/`): Each skill is a complete hedge fund persona with personality, philosophy, KPIs, and self-evaluation. Skills are exchange-agnostic — they work with any connected exchange.
- **Connectors** (`connectors/`): Exchange MCP servers that bridge Claude to exchange APIs. Cube ships built-in. Others install via npm.
- **Shared Libs** (`lib/`): Technical indicators, financial math, formatting utilities.

## Development Workflow

- **Requirements**: Node >= 20, ES modules (`"type": "module"`)
- **TypeScript**: Strict mode, ES2022 target, Node16 module resolution
- **Build**: `npm run build` — compiles Cube, Robinhood, Alpaca, and CCXT MCP server workspaces
- **Dev**: `npm run dev` — runs Cube MCP server with watch
- **Typecheck**: `npm run typecheck` — runs `tsc --noEmit` across project
- **Test**: `cd connectors/cube/mcp-server && npm test` — vitest (auth, signing, indicators, format, REST orders, WebSocket, credential store, device auth, integration)
- **Install agents**: `npx ai-fund install` (all), `npx ai-fund install <role>` (one), `npx ai-fund list` (show available)

### CI/CD

GitHub Actions runs on every push to `main` and on every PR (`.github/workflows/test.yml`):

1. `npm ci` — install dependencies
2. `npm run typecheck` — TypeScript strict check
3. `npm test --workspace=connectors/cube/mcp-server` — vitest suite

PRs that fail CI will not be merged. Fix locally before pushing.

### Validation After Changes

After every code change, run the following before considering the work done:

1. **Typecheck**: `npm run typecheck` — must pass with zero errors
2. **Unit tests**: Run relevant test suites; fix any failures before committing
   - Cube: `cd connectors/cube/mcp-server && npm test`
   - CCXT: `cd connectors/ccxt/mcp-server && npx vitest run`
3. **Update docs**: If your change affects architecture, commands, agent categories, shared libraries, or exchange support, update `CLAUDE.md` and `README.md` to reflect the new state

### Task Completion Checklists

- **Bug fix**: Write a test reproducing the bug + fix + typecheck + full test suite green
- **New skill**: Copy `skills/_template/SKILL.md` + fill all required sections + update agent categories in `CLAUDE.md` and `README.md`
- **New connector**: MCP server + tool namespace docs + update supported exchanges table in `CLAUDE.md` and `README.md`
- **Library change** (`lib/`): Add/update unit tests + verify all importing code still works if function signatures change
- **Command change** (`.claude/commands/`): Update commands list in `CLAUDE.md` and `README.md`

## Code Conventions

- **Languages**: TypeScript and Rust only. No Python, no JavaScript-without-types, no shell scripts beyond simple CLI wrappers. New connectors must be TypeScript or Rust — no exceptions.
- **Indentation**: 2 spaces, no tabs
- **Quotes**: Single quotes in TypeScript
- **File naming**: `kebab-case.ts` for files, `kebab-case/` for directories
- **TypeScript naming**: `PascalCase` for types/interfaces/classes, `camelCase` for functions/variables/parameters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cubexch/ai-fund](https://github.com/cubexch/ai-fund) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
