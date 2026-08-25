---
trigger: always_on
description: This file is read automatically by Claude Code at session start. It captures
---

# Claude Code — project memory for Nitro-V3

This file is read automatically by Claude Code at session start. It captures
the conventions and current state of this branch so a new session can hit
the ground running.

## TL;DR

This client carries a long-running React 19.2 modernization: React 19
idioms + supporting infrastructure (TanStack Query, Zustand, Vitest,
React Compiler, error boundaries), god-hook splits, and logic-bug audits.

**Working base is now `main`** (tracking `duckietm/Nitro-V3`). The earlier
`feat/react19-modernization` long-running branch was superseded — feature
work now ships as small focused PRs against `duckietm:Dev`, staged through
Dev then merged to main. (`feat/react19-modernization` still exists on the
fork as backup; do not force-push it.)

**Navigator modernization landed** (merged to main 2026-05-28, PRs
#168/#169/#170): the 492-line `useNavigator` god-hook was split into
`useNavigatorStore` + `useNavigatorData`/`useNavigatorUiState`/
`useNavigatorSearch` filters (wired-tools layout), door lifecycle extracted
to `src/hooks/rooms/widgets/useDoorState.ts`, 9 UI flags moved to a Zustand
`navigatorUiStore`, search migrated to a query hook, and 5 sub-views wrapped
in `WidgetErrorBoundary`. **Caveat**: duckietm patched `useNavigatorSearch`
post-merge (`05d71dd1`) — see the `useNitroQuery` fragility note below.

When syncing upstream, expect conflicts in `App.tsx` / `bootstrap.ts` /
`LoginView.tsx` on React 19 imports — always keep the modernized version.

Local-dev game assets are served by a small Vite plugin (`sirv` middleware
mounted on `/nitro-assets` and `/swf`, reading from
`E:\Users\simol\Desktop\DEV\Nitro-Files`) — NOT by symlinking inside
`public/`. The symlink path triggers chokidar on ~177k files and the dev
server hangs for minutes on Windows. See `vite.config.mjs` and the
`.gitignore` note.

Detailed status, decisions, and next steps live in **`docs/ARCHITECTURE.md`** —
read that before starting anything non-trivial.

## Commands

| Goal | Command |
|---|---|
| Dev server | `yarn start` |
| Production build | `yarn build` |
| Serve production build | `yarn preview` (defaults to http://localhost:4173) |
| Lint | `yarn eslint` |
| Type-check (TS 7 native, fast) | `yarn typecheck` |
| Test (Vitest, once) | `yarn test` |
| Test (watch) | `yarn test:watch` |

## Setup walkthrough

1. **Clone the renderer SDK as a sibling of this repo.**
   `vite.config.mjs` resolves the `@nitrots/*` aliases against
   `../Nitro_Render_V3` (preferred) or `../renderer` (legacy). If neither
   exists, the dev server and build now fail fast with a message
   pointing here.

   ```sh
   cd ..                            # parent of Nitro-V3
   git clone <renderer-repo> Nitro_Render_V3
   cd Nitro_Render_V3 && yarn install
   ```

2. **Install client deps.**
   ```sh
   cd ../Nitro-V3
   yarn install
   ```

3. **Materialize the runtime configuration.** `public/configuration/`
   ships `.example` files. Copy the ones you need without the `.example`
   suffix and point them at your game server (websocket URL, asset
   base URL, UI texts, etc.). The dev server doesn't fail if these are
   missing but the client renders a blank/error screen at runtime.

4. **Run.**
   - Dev: `yarn start` (Vite, HMR, includes the renderer source).
   - Production preview: `yarn build && yarn preview`.

The renderer SDK (`@nitrots/nitro-renderer`) is consumed via a filesystem
link to a sibling working tree — `../Nitro_Render_V3` (preferred) or
`../renderer` (legacy). Without it, `yarn typecheck` reports TS2307 across
the codebase — that's expected on a sandbox without the renderer, not a
regression.

## Runtime verification

This client can't be driven **headless** — the room renders via PixiJS/WebGL,
which hangs in a headless browser, and login needs an SSO session. To verify a
change at runtime, use the **Claude-in-Chrome extension** against the user's
real, logged-in `localhost:5173` session: it shares the browser's SSO cookie
(auto-login) and uses the real GPU. Toolbar buttons are canvas (not in the
accessibility tree) — locate them by screenshot/zoom, then click by coordinate.
The `preview_*` (headless) tools can't bind 5173 here and won't get past
login/WebGL, so they're not usable for in-game verification.

## Stack snapshot

- React 19.2.5, `react-dom` 19.2.5, `@types/react` 19.2.x.
- TypeScript: TS 6 for build, **TS 7 native preview** (`@typescript/native-preview`,
  invoked via `tsgo`) for the `typecheck` script.
- Vite 8 + `@vitejs/plugin-react` 6 + `babel-plugin-react-compiler` 1.0.
- ESLint 10 + `typescript-eslint` 8 + `eslint-plugin-react-hooks@7` +
  `eslint-plugin-react-compiler`.
- TanStack Query 5 (`@tanstack/react-query` + devtools).
- Zustand 5.
- Vitest 3 + jsdom + `@testing-library/react` + `@testing-library/jest-dom`.
- `react-error-boundary` 6.

## Layout convention (DO NOT CHANGE)

Established by the team and recorded in `docs/ARCHITECTURE.md` proposal #3
(rejected the `src/features/` alternative). Stay on this layout — every PR
that violates it will need to be reworked.

```
src/components/<area>/<feature>/         → views (.tsx only)
  e.g. src/components/room/widgets/doorbell/DoorbellWidgetView.tsx


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckietm/Octane](https://github.com/duckietm/Octane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
