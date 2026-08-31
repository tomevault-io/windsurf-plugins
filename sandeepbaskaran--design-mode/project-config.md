---
trigger: always_on
description: enables in this state.
---

# CLAUDE.md — Project guide for Design Mode

Project-level instructions for anyone (or any AI assistant) working
on Design Mode. Personal/per-maintainer workflow preferences live
outside the repo — this file is purely about the project.

If you're a contributor using Claude Code (or any CLAUDE.md-aware
tool), these rules are read automatically at session start.

---

## Workspaces

- `packages/extension` — Chrome MV3 side-panel extension (the main
  surface). TypeScript, Vite, no framework.
- `packages/mcp-local` — Local MCP companion + WebSocket bridge.
  Node.js, TypeScript.
- `packages/mcp-cloud` — Hosted MCP relay (Vercel-deployable).
- `packages/shared` — Types, message schemas, constants. Re-exported
  via `@shared/...`.
- `website/` — Next.js 15 / React 19 / Tailwind 4 / shadcn-ui
  (new-york) marketing + docs + interactive demo site. Light mode
  only; the `dark:` variants still present in shadcn primitives
  are inert (no `.dark` class is ever added to the root).

## Default build commands

- **One build, one `dist/`, one manifest — serves BOTH Chrome and
  Firefox.** Each browser reads its own manifest keys and ignores the
  other's; the JS detects the browser at **runtime**
  (`src/platform/target.ts` → `IS_FIREFOX`, UA + `sidebarAction`
  sniff). There is no separate Firefox build or `dist-firefox/`.
- **Build the extension:** `node build.mjs` from `packages/extension/`
  (NOT from the repo root — `build.mjs` lives inside the package).
  → `dist/`.
- **Single merged manifest** (`public/manifest.json`) carries both
  browsers' keys: Chrome uses `side_panel` + `sidePanel` permission +
  `background.service_worker`; Firefox uses `sidebar_action` +
  `background.scripts` + `browser_specific_settings` (Gecko id
  `sandeepbaskaran98@gmail.com`, `strict_min_version` 121.0, and
  `data_collection_permissions: { required: ["none"] }` — AMO rejects new
  submissions without it). The
  `background` block carries **both** `service_worker` and `scripts`
  (each browser picks its own). Don't split this into per-target
  manifests.
- **API layer:** the extension calls the promise-based `browser.*`
  namespace everywhere (webextension-polyfill on Chrome, native on
  Firefox — imported first in each entry via `src/platform/polyfill.ts`).
  `browser` is typed as `typeof chrome`. A handful of Chrome-only APIs
  stay on the `chrome` global: `chrome.sidePanel` (behind the
  `platform/panel.ts` adapter), `chrome.storage.session.setAccessLevel`,
  `chrome.extension.isAllowedFileSchemeAccess`. Firefox's
  `sidebarAction` is reached via a cast in `platform/panel.ts`.
- **Pop-out floating window + Picture-in-Picture are Chrome-only**
  (Firefox has neither `sidePanel` nor Document PiP) — those controls
  are gated behind `!IS_FIREFOX` at runtime (present in the shared
  bundle but dormant on Firefox; the docked sidebar is Firefox's sole
  surface). Alt+D opens the panel on both: Chrome via the side panel,
  Firefox via `sidebarAction.open()` called synchronously in the
  command handler.
- **`IS_FIREFOX` (`src/platform/target.ts`) is the single flag for any
  Chrome/Firefox-divergent behaviour** — UI copy, links, or feature
  gating. Always branch on it; never add ad-hoc `navigator.userAgent`
  checks elsewhere. Current divergences that use it: pop-out / PiP
  (hidden on FF), the eyedropper "Pick" button (hidden on FF — no
  EyeDropper API), the side-panel open adapter (`platform/panel.ts`),
  the keyboard-shortcut open path (`background/index.ts`), the
  file-access settings button (`about:addons` vs `chrome://extensions`),
  and the Contribute panel's "Review" link + share text (AMO vs Chrome
  Web Store). New browser-specific behaviour goes through this flag.
- **Build the website:** `npm run build` from `website/`.
- **Repo-wide gate:** `npm run verify` from repo root — runs
  `scripts/prepublish-check.mjs` which chains: build extension →
  bundle integrity → manifest sanity (asserts BOTH Chrome + Firefox
  keys) → `web-ext lint` (0 errors required; the ~30 warnings —
  `innerHTML`, Chrome-only `UNSUPPORTED_API`, the ignored `sidePanel`
  permission, `BACKGROUND_SERVICE_WORKER_IGNORED`, and the advisory
  `KEY_FIREFOX_UNSUPPORTED_BY_MIN_VER` for `data_collection_permissions`
  — are expected) →
  MCP local checks → build local MCP → bundle integrity → build
  website → export integrity. CI uses the same command.
- **Two store zips, identical content, distinct names** (both under
  `packages/extension/`, both gitignored): `design-mode-extension.zip`
  (Chrome Web Store, `npm run package:extension`) and
  `design-mode-addon.zip` (Firefox / AMO + local `about:debugging`
  testing, `npm run package:extension:firefox`).
  `npm run package:extension:all` builds once and writes both.
  They're byte-for-byte the same dist — the split is purely so each
  store gets a recognisably-named file. Replace the existing zips;
  never produce per-version copies like `design-mode-v1.0.2.zip`.
  `npm run lint:extension` builds + runs `web-ext lint`.
- **TypeScript typecheck has a known pre-existing failure** because
  `packages/shared` lacks `composite: true`. `npm run verify` does
  NOT run `tsc` on shared, so CI is fine. Don't "fix" this
  incidentally.

## Code conventions

- **No comments describing WHAT the code does** — well-named

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SandeepBaskaran/design-mode](https://github.com/SandeepBaskaran/design-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
