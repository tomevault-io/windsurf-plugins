---
trigger: always_on
description: npm test                              # run all tests (output buffered until done)
---

# Copilot Instructions for TermBeam

## Build, Test, and Lint

```bash
npm test                              # run all tests (output buffered until done)
node --test 'test/server/*.test.js' 'test/cli/*.test.js' 'test/utils/*.test.js' test/integration.test.js  # all tests with streaming output (preferred for dev/CI agents)
node --test test/server/auth.test.js  # run a single test file
npm run test:coverage                 # tests + coverage (c8, 92% threshold)
npm run lint                          # syntax-check with node --check
npm run format                        # format with Prettier
npm run dev                           # start with auto-generated password
npm start                             # start with defaults
```

> **Agent note:** Prefer the `node --test` glob command over `npm test` when you need streaming output. The `npm test` script wraps `node --test` in `execFileSync` which buffers all output until completion — this makes it look like tests are hanging when they're actually running fine. The direct command gives real-time feedback. Use quoted glob patterns (`'test/server/*.test.js'`) — bare directory paths (e.g. `test/server`) cause `MODULE_NOT_FOUND` errors.

Pre-commit hooks (Husky + lint-staged) auto-format and syntax-check staged files.

### Testing Best Practices

**Suite overview:** 575+ tests, ~17s total. Tests run in parallel child processes via Node's built-in test runner. Most files run in <1s; `integration.test.js` (~17s) and `test/cli/service.test.js` (~9s) are the slow outliers.

**Slow tests and why:**

- `integration.test.js` — uses real PTY servers, has polling loops and a 7s sleep for git cache invalidation
- `test/cli/service.test.js` — heavy `require.cache` manipulation and `process.exit` mocking

**Test isolation rules (critical for reliability):**

- **`process.exit` mocks** — always restore in `afterEach`, never inline. Failing to restore breaks subsequent tests.
- **`console.log`/`console.error` mocks** — same rule: restore in `afterEach`.
- **`test/cli/service.test.js`** — uses `loadServiceWithMocks()` pattern; always call `.restore()` in `afterEach`.
- **`test/server/sessions.test.js`** — manipulates `require.cache` for node-pty mocking; clear cache between tests.
- **`test/cli/resume.test.js`** — uses `TERMBEAM_CONFIG_DIR` env var pointing to a temp directory for isolation.
- **WebSocket connections** — close in `finally` blocks or `after()` hooks to prevent connection leaks.
- **Windows temp directory cleanup** — on Windows, node-pty ConPTY holds directory locks after `pty.kill()`. Use `await safeCleanup(dir)` (async `fs.promises.rm` with `maxRetries`) instead of `fs.rmSync` in `after()` hooks and `finally` blocks when the temp dir was used as a PTY CWD. See the `safeCleanup()` helper in `test/server/routes.test.js`.
- **`configDir` isolation** — every test that calls `createTermBeamServer({ config })` MUST set `configDir` (or `TERMBEAM_CONFIG_DIR` env var) to a fresh `mkdtempSync` temp directory. Without this, the server reads the developer's real `~/.termbeam/prefs.json` and may auto-spawn workspace sessions with a different `cwd`, silently breaking assertions about the default session. `routes.test.js`'s `startServer()` is the canonical example.
- **`--test-force-exit`** — `npm test` passes this flag so dangling Windows ConPTY threads can't keep the worker alive past the 180 s per-file timeout. Tests that legitimately need to leak resources for assertions should be flagged in the suite description.

**Port isolation:** Integration tests use port `0` (OS-assigned random port) to avoid conflicts. Never hardcode ports in tests.

### Playwright E2E

```bash
npx playwright test                   # run e2e tests (chromium, sequential)
```

E2E tests live in `test/e2e-*.test.js` and are excluded from `npm test`. See `playwright.config.js` for retries, reporters, and timeouts.

## Architecture

TermBeam is a Node.js CLI tool that exposes a local PTY (pseudo-terminal) over HTTP + WebSocket, with a mobile-optimized browser UI.

**Server flow:** `bin/termbeam.js` → `src/server/index.js` (orchestrator) creates an Express app + WebSocket server, wiring together modules from `src/server/`, `src/cli/`, `src/tunnel/`, and `src/utils/`.

**`src/server/`** — HTTP/WS server core:

- `index.js` — orchestrator: creates Express app + WebSocket server
- `routes.js` — Express routes for API (`/api/sessions`, `/api/auth`) and pages (`/terminal`, `/login`)
- `auth.js` — password auth, token cookies, rate limiting, login page
- `sessions.js` — `SessionManager` class wrapping `node-pty` lifecycle (create/list/delete/shutdown), sessions stored in a `Map`, clients tracked per session in a `Set`
- `websocket.js` — handles WebSocket messages (`attach`, `input`, `resize`, `output`, `exit`)
- `preview.js` — local preview proxy for forwarding requests to a port

**`src/cli/`** — CLI subcommands & tools:

- `index.js` — parses CLI flags and env vars into a config object
- `client.js` — WebSocket terminal client: raw mode stdin/stdout piping, Ctrl+B detach, resize (SIGWINCH), scrollback replay. Used by `resume.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dorlugasigal/TermBeam](https://github.com/dorlugasigal/TermBeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
