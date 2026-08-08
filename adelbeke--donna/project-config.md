---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Donna is an open source GitHub companion — filter, prioritise, and track review status across all your repositories. The **Electron app** (macOS) is the sole functional product: full features including the Branches tab, with auth and all GitHub calls delegated to the local `gh` CLI — no token is ever stored.

The React renderer previously also shipped as a browser-only web app (GitHub Pages at `adelbeke.github.io/donna`), authenticating with a classic PAT kept in `localStorage`. That web app has been retired — GitHub Pages now serves a static `web-deprecated/index.html` pointing visitors to the latest release, built and deployed independently of the React app (see `.github/workflows/deploy.yml`). The renderer source no longer contains a web entry point (`WebContainer`/`AuthPage` were removed); `authStore` remains shared infrastructure used by the Electron app's `gh-cli` sentinel token flow.

## Commands

```bash
npm run dev            # web renderer on http://localhost:5173/donna/
npm run dev:electron   # Electron window (expects the vite dev server above to be running)
npm run lint           # ESLint (flat config)
npm test               # vitest watch
npm run test:run       # vitest once  — run a single file: npm run test:run src/features/pull-requests/lib/prUtils.test.ts
npm run build          # web build: tsc -b && vite build
npm run build:electron # electron-vite build && electron-builder (produces signed .dmg)
```

CI (`.github/workflows/ci.yml`) runs `lint`, `test:run`, then `build` on every PR — all three must pass.

## Main Features

**Pull Requests view** (`prs`) — three sections selected via the `PRSectionsTabs` sidebar tabs, each a different GitHub search (`buildSearchQuery`):
- **Review requested** / **My PRs** (authored) / **Reviewed**.
- Per-PR actions on each `PRCard`: **star** (top-priority, pinned above the list and persisted), **hide** (dims + filtered out unless "Hidden" toggle is on), **copy PR link**, open externally.
- **Mute authors**: free-form patterns (e.g. `dependabot`) that filter out their PRs.
- Filters: by repository/org (checkboxes in the `SettingsModal` gear-icon popover, shown only when >1 repo loaded), title search (above the list), show/hide **drafts**, show/hide **hidden** (`VisibilityToggles` in the list header).
- Each card shows repo, author, `#number`, opened/updated ages, diff size (`+/-`), draft/hidden badges, **my review-state** badge, a **CI checks** badge that opens a `PRChecksModal` popover (lazy-loads per-check contexts), and a **conflict** badge when `mergeable === 'CONFLICTING'`. In the authored section, cards also show grouped **reviewer avatars** (approved / changes-requested / commented / pending).
- Paging: pages auto-load in the background (capped at `MAX_PAGES = 10`); a spinner in the header badge shows while more pages are fetching.
- **Focus refresh**: when the window regains focus, `useFocusRefresh` (`hooks/useFocusRefresh.ts`) snapshots the current PR IDs and triggers a refetch. If new PRs appear, a `NewPRsBadge` is shown above the list; the list stays frozen at the pre-focus snapshot until the badge is dismissed or clicked. The `authored` section is excluded from this detection. Local mutations (star/hide) pass through the snapshot filter so they are always reflected immediately.

**Branches view** (`branches`, **Electron-only**) — `BranchDashboard` renders a branch-name search box plus `BranchList`, which reads *local* git repos the user adds via a native directory picker:
- Lists branches per repo with **worktree detection**, a **dirty-state** dot for uncommitted changes, and the **linked open PR** (matched by `repo/headRefName`).
- One-click copy of `git switch <branch>` and `cd <worktree-path>`.

**App-wide**: light/dark theme toggle, OTA self-update banner (Electron), version in the footer.

## Repository layout

```
electron/            Electron main + preload (Node side; the only place that shells out)
  main.ts            ipcMain handlers: gh:graphql/rest/installed, branches/worktrees list, dialog, updater
  preload.ts         contextBridge → window.electronAPI (typed in src/types/electron.d.ts)
src/
  App.tsx            QueryClient setup, always mounts AppContainer
  main.tsx           React entry
  index.css          Tailwind v4 @theme tokens + [data-theme="light"] overrides
  containers/        AppContainer.tsx (Electron: gh CLI auth probe, error screen, mounts DashboardPage)
                     DashboardPage.tsx (navbar + view switch)
  features/          Feature slices — each has components/, queries/, stores/ + exports.ts barrel
    auth/            authStore (token + user), backing AppContainer's gh-cli sentinel token flow
      stores/                authStore.ts (Zustand, persisted: token + user) + authStore.test.ts
      exports.ts             public surface: { useAuthStore }
    branches/        Electron-only Branches tab
      components/BranchDashboard/   BranchDashboard.tsx (search input + BranchList)
      components/BranchList/        BranchList.tsx + BranchList.test.tsx, BranchCard/ (BranchCard.tsx)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adelbeke/donna](https://github.com/adelbeke/donna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
