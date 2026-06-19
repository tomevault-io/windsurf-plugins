---
trigger: always_on
description: Electron + React + TypeScript desktop chat app wrapping the Claude Agent SDK. The main process spawns the SDK, streams events to the renderer over a zod-validated IPC channel, and asks the user for permission whenever the SDK wants to invoke a tool not covered by the bundled allow list.
---

# Studio Write — agent guide

Electron + React + TypeScript desktop chat app wrapping the Claude Agent SDK. The main process spawns the SDK, streams events to the renderer over a zod-validated IPC channel, and asks the user for permission whenever the SDK wants to invoke a tool not covered by the bundled allow list.

## Run & test

The user normally runs `npm start` in a separate terminal. `npm start` wraps `electron-forge start` via `scripts/dev.mjs` and exposes a reload socket (under `os.tmpdir()`, keyed by a hash of the project root so parallel worktrees don't collide) so agents can restart the main process without the user typing `rs`. Preload/renderer edits HMR automatically; for main-process or `resources/` edits, run `npm run reload` — it blocks until a fresh `bootId` lands in `.vite/dev-boot.json`, and the marker is only written after the renderer finishes loading, so exit 0 means the app is back up _and_ the Playwright MCP CDP target is attachable (no mid-reload reattach races). The socket lives outside `.vite/` because electron-forge's vite plugin wipes that dir on startup, which would unlink a socket file placed there.

Before any Playwright MCP call or `npm run reload`, run `npm run ensure-dev`. It's idempotent: if the dev server is already up it exits immediately; if not, it opens a visible Terminal.app window running `npm start` (so the user can watch forge/Vite output), or on non-macOS falls back to a detached process logging to `.vite/dev.log`. Either way it blocks until the new boot marker lands. Prefer this over asking the user to start npm themselves.

-   `npm start` — Electron + Vite HMR via `scripts/dev.mjs` (user's responsibility)
-   `npm run ensure-dev` — start `npm start` detached if it isn't already running; blocks until CDP is live
-   `npm run reload` — restart the Electron main process; blocks until the new process is live
-   `npm test` — runs `test:unit` then `test:e2e`.
-   `npm run test:unit` — Vitest over `tests/unit/` (pure functions, sub-second).
-   `npm run test:unit:watch` — Vitest in watch mode.
-   `npm run test:e2e` — Playwright over `tests/e2e/`. `tests/global-setup.ts` unconditionally runs `TEST_BUILD=1 npm run package` before the suite. Packaging takes ~5s; we don't cache it — a prior marker/fuse-check scheme kept reusing stale builds and caused flaky failures that only cleared after `rm -rf out`.
-   `npm run lint` / `lint:css` / `format` — WordPress-flavored ESLint, Stylelint, wp-prettier.

E2E specs that hit the agent need `ANTHROPIC_API_KEY` in `.env` or the shell. All e2e specs seed an isolated userData dir via `STUDIO_WRITE_USER_DATA_DIR` + a linked tmp folder (see `tests/helpers/linked-projects.ts`) so runs don't touch the real app's state.

## Visual inspection via Playwright MCP

Unpackaged builds expose CDP on a per-worktree port derived from the project root (`main.ts`, gated by `! app.isPackaged`). `.mcp.json` runs `scripts/playwright-mcp.mjs`, which recomputes the same port from `process.cwd()` and spawns `@playwright/mcp` against it, so parallel worktrees don't cross-wire. The Vite renderer port is hashed the same way (`vite.renderer.config.ts`) with `strictPort: true`, so each worktree has a stable renderer URL that can't drift when sibling worktrees boot in different orders. Both values are written into `.vite/dev-boot.json` (`cdpPort`, `rendererUrl`) — read that if you need to point DevTools or a browser at the live app yourself. Agents drive the live dev window via `browser_click` / `browser_type` / `browser_take_screenshot` / `browser_run_code`.

Quirks:

-   **Screenshots lose the backdrop.** The window uses transparent bg + macOS vibrancy; CDP captures web contents only, so transparent pixels come back white. Inject `html, body { background: ... }` before shooting, remove after.
-   **Dark mode needs `page.emulateMedia({ colorScheme: 'dark' })`** via `browser_run_code` — `matchMedia` reflects Chromium's emulation, not the OS.
-   **Reloads drop the session.** `Target ... has been closed` on the next call is expected; retry and the MCP re-attaches.
-   **`browser_navigate` hijacks the app window.** Navigating to the CDP port replaces the app with the CDP listing page; recover with `browser_navigate(<rendererUrl from .vite/dev-boot.json>)`. The Vite renderer port is per-worktree — don't hardcode 5173.
-   **Save screenshots under `.playwright-mcp/`** — `/tmp` is outside the MCP's allowed roots. Don't commit `page-*.png` / `app-*.png` (they land in the repo root).

### Fast verification via `window.__sw`

Every MCP tool call is a ~1–2s round-trip, so blind `browser_snapshot` + `browser_wait_for(time)` between steps is expensive. Use the dev-only `window.__sw` surface (installed by `App.tsx` when the renderer runs from `http://localhost`):

-   `__sw.isStreaming()` — any assistant bubble has `data-streaming="true"`.
-   `__sw.hasPendingPermission()` — `[data-testid=permission-prompt]` is on screen.
-   `__sw.isIdle()` — neither of the above.

Two rules for agent-driven verification:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Automattic/studio-write](https://github.com/Automattic/studio-write) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
