---
trigger: always_on
description: Purpose: OpenCode authentication plugin for Atlassian Rovo Dev + optional local proxy that
---

# Agent Notes: opencode-rovodev-auth

Purpose: OpenCode authentication plugin for Atlassian Rovo Dev + optional local proxy that
exposes an OpenAI-compatible `/v1/*` API backed by Rovo Dev's `/v3/*` serve mode.

## Tooling / Environment

- Node: >= 20 (see `package.json#engines`)
- Package manager: npm (`package-lock.json` present; use `npm ci` to install)
- TypeScript: `tsc` (build + typecheck); target ES2022, moduleResolution `bundler`
- Optional runtime for proxy: Bun (`rovodev-proxy.ts` is executed by bun, not tsc)

## Common Commands

```bash
npm ci                    # install (use npm install only when changing deps)
npm run build             # compile src/ → dist/
npm run typecheck         # tsc --noEmit (no output on success)
```

### Run Local Rovo Dev Proxy (dev utility)

Prereqs: `bun` installed + Atlassian CLI (`acli`) authenticated for rovodev.

```bash
# 1) Start Rovo Dev serve mode
acli rovodev serve 8123 --disable-session-token

# 2) Start proxy (default: rovodev 8123, proxy 4100)
bun rovodev-proxy.ts
bun rovodev-proxy.ts --rovodev-port 8123 --proxy-port 4100
```

Windows helper: `start-rovodev.bat` (launches both processes).

Smoke checks:
```bash
curl http://localhost:4100/health
curl http://localhost:4100/v1/models
```

## Lint / Format / Tests

- Lint: not configured.
- Formatting: not enforced by tooling; match existing style.
- Tests: no test runner configured. Do NOT claim tests exist unless you add them.

### Adding Tests (If/When Needed)

Pick ONE runner and wire it into `package.json`. Suggested single-test commands:

```bash
# Node built-in
node --test test/my-feature.test.ts
node --test --test-name-pattern "parses SSE" test/my-feature.test.ts

# Vitest
npx vitest test/my-feature.test.ts
npx vitest -t "parses SSE" test/my-feature.test.ts

# Bun
bun test test/my-feature.test.ts
bun test --test-name-pattern "parses SSE" test/my-feature.test.ts
```

## Repository Layout

```
src/
  index.ts                          # library entry point (re-exports plugin)
  plugin.ts                         # OpenCode plugin: auth hook + URL rewriting
  runtime/
    server.ts                       # createRuntimeServer(): routes /v1/* requests
    backend/
      rovo-serve-driver.ts          # RovoServeDriver: queued request → /v3/* Rovo Dev
      types.ts                      # BackendDriver, BackendTurnRequest, etc.
    diagnostics/
      logger.ts                     # logRequestSummary, logWarning helpers
    openai/
      chat.ts                       # POST /v1/chat/completions handler
      models.ts                     # GET /v1/models handler
      responses.ts                  # POST /v1/responses handler
    policy/
      capability-policy.ts          # ROVO_SERVE_CAPABILITIES constant
      model-policy.ts               # model list/validation
    session/
      message-compiler.ts           # normalizeIncomingMessages, formatMessages, etc.
      response-builder.ts           # OpenAI response shape builders
      session-store.ts              # session state helpers
    stream/
      sse-mapper.ts                 # maps backend SSE events → OpenAI SSE events
      sse-parser.ts                 # parseSSELines, extractBackendText, isBackendStreamEnd
rovodev-proxy.ts                    # Bun entrypoint (not included by tsconfig)
dist/                               # tsc output — never hand-edit
```

**Important:** `rovodev-proxy.ts` is NOT included in `tsconfig.json`. `npm run build` and
`npm run typecheck` only validate `src/**/*`.

## TypeScript + ESM Style Guide

### Module System / Imports

- Package is ESM (`"type": "module"`). Use `import`/`export` only; never `require`.
- Use `.js` extensions in relative imports in TS source:
  - Good: `import { foo } from "./bar.js";`
  - Bad: `import { foo } from "./bar";` (breaks Node ESM resolution)
- Prefer `import type { ... }` for type-only imports.
- Import order (blank line between groups):
  1. Node built-ins
  2. External packages
  3. Local modules

### Formatting

- Indentation: 2 spaces.
- Strings: double quotes.
- Semicolons: required.
- Trailing commas: in multiline objects/arrays.
- Wrap long ternaries and chains for readability.

### Types

- `strict: true` is enabled; all new code must pass strict typecheck.
- Prefer `unknown` + narrowing over `any`.
- Use `any` only at parsing boundaries (e.g. raw JSON/SSE payloads); isolate it.
- Use `as const` for narrow string literal fields.
- Prefer small named helper types over repeated large inline object types.

### Naming Conventions

- Types / interfaces / classes: `PascalCase`
- Functions / variables: `camelCase`
- Module-level constants: `SCREAMING_SNAKE_CASE` (e.g. `ROVODEV_PROVIDER_ID`)
- Abbreviations only when domain-standard: SSE, URL, HTTP, SDK.

### Error Handling

- Treat all network responses, JSON payloads, and SSE data as untrusted.
- Wrap `req.json()`, upstream `fetch()`, and SSE JSON parsing in `try/catch`.
- Catch with `catch (err: unknown)` and narrow before reading `err.message`.
- Structured error response shape:
  - `{ error: { message: string, type: "invalid_request_error" | "proxy_error" } }`
- Status codes: `400` bad input, `404` unknown route, `502` upstream/proxy failure.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lutfi238/opencode-rovodev-auth](https://github.com/lutfi238/opencode-rovodev-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
