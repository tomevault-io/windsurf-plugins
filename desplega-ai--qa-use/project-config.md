---
trigger: always_on
description: QA-Use is an MCP server for browser automation and QA testing using Playwright, integrating with desplega.ai. Landing site at https://qa-use.dev (deployed from `landing/`).
---

# CLAUDE.md

QA-Use is an MCP server for browser automation and QA testing using Playwright, integrating with desplega.ai. Landing site at https://qa-use.dev (deployed from `landing/`).

## Project map

```
src/
├── cli/               # Unified `qa-use` CLI
│   ├── index.ts       # CLI entry point
│   └── commands/      # api, app-config, app-context, browser, data-asset,
│                      # issues, persona, suite, test, docs, setup, usage,
│                      # info, install-deps, mcp, update, tunnel, doctor
├── index.ts           # MCP stdio entry (direct invocation)
├── server.ts          # Main MCP server, tools, session management
├── http-server.ts     # HTTP/SSE transport
├── tunnel-mode.ts     # Persistent WebSocket tunnel mode
└── types.ts

lib/
├── api/               # desplega.ai API client
├── browser/           # Playwright browser management
├── env/               # Environment & config loading
└── tunnel/            # Localtunnel wrapper
```

## Tech stack

Node.js 20+ / TypeScript / **bun** runtime. `@modelcontextprotocol/sdk` (stdio + HTTP/SSE), Playwright (Chromium), `@desplega.ai/localtunnel`. Tests run with bun.

<important if="you need to run commands to install, build, test, lint, format, typecheck, or run the CLI">

**Always use `bun`, never `npm` or `yarn`.**

| Command | What it does |
|---|---|
| `bun install` | Install dependencies |
| `bun run build` | Build TypeScript |
| `bun run dev` | Development with hot reload |
| `bun test` | Run tests |
| `bun run lint:fix` | Fix linting issues |
| `bun run format` | Format code |
| `bun run typecheck` | Type check |
| `bun run check:fix` | Lint + format fix (run before committing) |
| `bun run cli <subcommand>` | Run the `qa-use` CLI from source |
| `bun run scripts/e2e.ts` | E2E regression script (see e2e block below) |

</important>

<important if="you have made code changes and are about to finish a phase, hand off, or commit">

After **any** code changes, run:

```bash
bun run check:fix
```

When implementing a multi-phase plan, run this after each phase before proceeding.

</important>

<important if="you are configuring qa-use, setting credentials, or debugging API/region connection issues">

Required: `QA_USE_API_KEY=xxx`. Optional: `QA_USE_REGION=us|auto` (default `auto`), `QA_USE_API_URL=xxx` to override the endpoint.

A `~/.qa-use.json` config file is also supported. **Env vars take precedence over the file.** The file may include a top-level `"tunnel": "auto" | "on" | "off"` (see tunnel block below).

For local backend work, the repo's `.qa-use.json` is pre-configured with `localhost:5005` and a valid API key — no extra env setup needed.

</important>

<important if="you are working on the MCP server, browser session lifecycle, or per-call timeouts in src/server.ts or src/http-server.ts">

- Three transport modes: **stdio** (default MCP), **HTTP/SSE** (web), **tunnel** (backend-initiated, persistent WebSocket — see `src/tunnel-mode.ts`).
- `BrowserSession` wraps browser + tunnel with a 30 min default TTL and auto-cleanup.
- Hard cap of **10 concurrent sessions**. Deadline is refreshed on every interaction.
- Per-MCP-call wall-clock budget is **25s** for timeout protection — long-running work must be chunked.
- The MCP global-browser path (`src/server.ts:450-603`) manages its own single `TunnelManager` and is **orthogonal** to the CLI tunnel-registry path.

</important>

<important if="you are mutating typed `variables:` on a test (CLI work, not test-author work)">

`qa-use test vars list | set | unset` is the imperative twin of the declarative
YAML editing path. It accepts either a positional `<file>` *or* `--id <uuid>`
(mutually exclusive). The local-file path mutates via yaml's Document API so
comments + key order survive; the `--id` path read-modify-writes via
`exportTest` + `importTestDefinition`. Sensitive-preserve: passing `--sensitive`
without `--value` keeps the stored value on an existing sensitive key — see
`plugins/qa-use/skills/qa-use/SKILL.md` (§ Test Variables).

The `TEST_VARIABLE_TYPES` / `TEST_VARIABLE_LIFETIMES` / `TEST_VARIABLE_CONTEXTS`
runtime arrays in `src/cli/lib/test-vars.ts` mirror the auto-generated unions
in `src/types/test-definition.ts`. A compile-time mutual-extends guard fails
`bun run typecheck` if you regenerate types via `bun run generate:types` and
the unions drift — update the arrays in lockstep.

</important>

<important if="you are adding or modifying TypeScript imports in this project">

Use `.js` extensions on relative imports (this is an ESM project) — e.g. `import { foo } from './bar.js'`, even when `bar.ts` is the source file.

</important>

<important if="you are adding logging or debug output in any code path that runs under MCP stdio">

Use `console.error()`, never `console.log()`. **stdout is reserved for the MCP protocol** — anything written there will corrupt the JSON-RPC stream.

</important>

<important if="you are modifying any browser command in src/cli/commands/browser/*.ts or the REPL in src/cli/commands/browser/run.ts">

CLI and REPL share the same functionality and **must stay in sync**:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [desplega-ai/qa-use](https://github.com/desplega-ai/qa-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
