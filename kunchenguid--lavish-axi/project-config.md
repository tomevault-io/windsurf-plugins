---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Commands

```sh
npm run check          # Run build, lint, format check, typecheck, and tests
npm run build          # Bundle dist/cli.mjs and copy chrome/design assets into dist
npm test               # node:test runner (test/*.test.js)
npm run lint           # ESLint over bin src test scripts
npm run format:check   # Prettier check
npm run typecheck      # tsc --noEmit (checkJs mode)
```

Run a single test file: `node --test test/server.test.js`.
Filter by test name: `node --test --test-name-pattern "createOpenOutput" test/cli-output.test.js`.

The `prepack` script runs `build` automatically, so publishing always ships a fresh bundle.

## Project Conventions

- Node 22+, ESM-only JavaScript (`"type": "module"`). No TypeScript source - `.js` files validated via TS `checkJs`.
- Use TDD for bug fixes and new features (see `test-driven-development` skill).
- Run `npm run check` before pushing.
- Do not hand-edit `CHANGELOG.md` or `.release-please-manifest.json` - release-please owns them.
- Human-authored PRs to `main` must go through [`no-mistakes`](https://github.com/kunchenguid/no-mistakes); CI enforces a deterministic signature in the PR body. See CONTRIBUTING.md.

## Architecture

Lavish Editor is a CLI + local HTTP server that opens agent-generated HTML artifacts in a browser, lets the user annotate elements or selected text ranges, and ships those annotations back to the agent over a long-polling API.

### Process model

The CLI (`bin/lavish-axi.js` -> `src/cli.js`) is the user-facing entry point.
The first command that needs the server spawns `lavish-axi server` as a **detached** background process (`src/cli.js:startServer`) and waits for `/health`, which returns `{ ok, app, version }`.
Subsequent CLI invocations reuse the running server only when its health version matches the current CLI version; stale servers are asked to `POST /shutdown`, and pre-handshake servers may be SIGTERM'd by port PID before the upgraded server is spawned.
Port defaults to 4387 (`LAVISH_AXI_PORT`).

State lives at `~/.lavish-axi/state.json` (override with `LAVISH_AXI_STATE_DIR`). All sessions across all projects share this one file, keyed by a sha256 prefix of the canonicalized file path - so the CLI never needs opaque session IDs; the canonical HTML path _is_ the identity (`src/session-store.js:sessionKey`).

### Request flow

1. `lavish-axi <file.html>` (`openCommand`) -> POST `/api/sessions` -> `SessionStore.upsertSession` -> server returns `http://localhost:PORT/session/<key>` and the CLI calls `open` to launch the browser.
2. The browser loads `GET /session/:key`, which serves a chrome page (`createChromeHtml`) containing an iframe pointing at `/artifact/:key/index.html`, a stylesheet at `/chrome.css`, and browser behavior at `/chrome-client.js`.
   The chrome client reads its session bootstrap from the `lavish-session` JSON script in the page.
3. The artifact route reads the HTML from disk and runs `injectLavishSdk` (`src/html-transform.js`) to append `<script src="/sdk.js?key=...">` and, unless the artifact opts out with `<meta name="lavish-design" content="off">`, inject `/design/daisyui.css`, `/design/tailwindcss-browser.js`, and `/design/daisyui-themes.css`.
4. Sibling assets resolve under `/artifact/:key/<path>`, sandboxed to the artifact's directory (`resolveArtifactAsset` rejects paths that escape via `..`), while packaged Tailwind/DaisyUI assets are served from `/design/:asset`.
5. User actions in the iframe `postMessage` to the chrome (queue prompts, request snapshot, end session). The chrome POSTs collected prompts to `/api/:key/prompts`, which queues them in the session store and emits a `feedback` event. Text selection prompts use `tag: "text"` and preserve a structured `target` with `type: "text-range"`, selected text, `commonAncestorSelector`, and start/end boundary anchors.
6. `lavish-axi poll <file.html>` (`pollCommand`) hits `GET /api/poll`. If queued prompts exist, it returns immediately; otherwise it long-polls on an `EventEmitter` until a `feedback` or `ended` event fires (default: no timeout - `--timeout-ms` is a test escape hatch).
7. `--agent-reply` posts a chat message into the session before polling, so the agent's reply renders in the browser conversation panel via the `/events/:key` SSE stream.

### Live reload

When a session opens, `chokidar` watches the artifact's directory (excluding `.git`, `node_modules`, `dist`, `build`, `.lavish-axi`).
Any file change emits a `reload` event, which the SSE endpoint pushes to the chrome page, which reloads the iframe.
During version-driven shutdown, the server sends a `chrome-reload` SSE event so open browser chromes wait for the replacement server and then reload the whole chrome page.
Hand-edited files in `dist/` won't trigger reloads.

### AXI integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/lavish-axi](https://github.com/kunchenguid/lavish-axi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
