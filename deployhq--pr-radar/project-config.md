---
trigger: always_on
description: PR Radar - a free browser extension (Manifest V3, Chrome + Firefox + Edge) that provides a unified PR dashboard for GitHub, GitLab, and Bitbucket. Shows CI status, unresolved comments, review state, deployment status, and sound/desktop notifications. No tab required — works in the background. Free, by DeployHQ.
---

# CLAUDE.md

## Overview

PR Radar - a free browser extension (Manifest V3, Chrome + Firefox + Edge) that provides a unified PR dashboard for GitHub, GitLab, and Bitbucket. Shows CI status, unresolved comments, review state, deployment status, and sound/desktop notifications. No tab required — works in the background. Free, by DeployHQ.

## Commands

```bash
npm run dev              # Chrome dev (watch mode)
npm run dev:firefox      # Firefox dev (watch mode)
npm run build            # Chrome production build (typecheck + vite build)
npm run build:chrome     # Chrome production build (explicit)
npm run build:firefox    # Firefox production build
npm run build:edge       # Edge production build
npm run typecheck        # TypeScript check only
npm run lint             # ESLint
```

### Loading locally

**Chrome**:
1. `npm run build`
2. Chrome > `chrome://extensions/` > Developer mode > Load unpacked > select `dist/`

**Firefox**:
1. `npm run build:firefox`
2. Firefox > `about:debugging#/runtime/this-firefox` > Load Temporary Add-on > select `dist-firefox/manifest.json`

**Edge**:
1. `npm run build:edge`
2. Edge > `edge://extensions/` > Developer mode > Load unpacked > select `dist-edge/`

To update: rebuild, then click refresh on the extension card (or remove + re-add for icon changes)

## Tech Stack

- **TypeScript**, **React 18**, **Tailwind CSS 3**
- **Vite 5** with `@crxjs/vite-plugin` for Chrome extension bundling
- **Manifest V3** (required for Chrome Web Store)

## Architecture

```
src/
  background/
    service-worker.ts            # Polling, badge updates, notifications, sound trigger
  popup/
    App.tsx                      # Main app shell with routing
    pages/
      Setup.tsx                  # Multi-platform connection (GitHub/GitLab/Bitbucket PAT auth, scope guide, connected state)
      Dashboard.tsx              # PR list with tabs (Mine/Review/All), cache-first rendering
      Settings.tsx               # Notifications, sound, polling, stale PR config, accounts, test button
      Repos.tsx                  # Watched repo selector with platform filter, select all, pin/fav stars, token scope callouts
    components/
      Header.tsx                 # Navigation header with extension icon + "by DeployHQ"
      PRItem.tsx                 # PR row: badges, diff stats, description preview, deployment URL, pinned star, stale/reviewed dimming
      CIBadge.tsx                # CI status badge (icon-only, tooltip with details + "broken by" on failure)
      PlatformIcon.tsx           # SVG icons for GitHub, GitLab, Bitbucket
      KeyboardShortcuts.tsx     # ? cheat sheet overlay (j/k nav, o open, 1-3 tabs, / search, r refresh)
      TriageSummary.tsx          # Urgency filter chip bar (icon+count chips with tooltips)
    utils/
      urgency.ts                 # Urgency classification, filter predicate, count computation, display metadata
    index.css                    # Tailwind imports + dark scrollbar styles
  shared/
    types.ts                     # TypeScript types (PullRequest, Platform, Message, UrgencyCategory, etc.)
    constants.ts                 # Status colors, platform labels, sound options
    storage.ts                   # Chrome storage wrapper (accounts, settings, repos, PR cache, CI statuses)
    api/
      github.ts                  # GitHub REST + GraphQL API (PRs, CI, reviews, threads, deployments, orgs, merge)
      gitlab.ts                  # GitLab REST API (MRs, CI pipelines, discussions, approvals, deployments, merge)
      bitbucket.ts               # Bitbucket REST API (PRs, pipelines, comments, participants, merge, workspaces)
      deployhq.ts                # DeployHQ REST API (projects, servers, deployments — opt-in integration)
public/
  offscreen.html                 # Offscreen document for audio playback (references offscreen.js)
  offscreen.js                   # Audio player (separate file required by MV3 CSP - no inline scripts)
  sounds/                        # Notification sounds: ding.mp3, bell.mp3, chime.mp3
icons/                           # Extension icons: 16/32/48/128px (radar dish on DeployHQ purple)
```

## API Integration

Uses PATs (Personal Access Tokens) — no backend needed. Setup page links pre-fill scopes.

### GitHub

- **Auth**: Classic PAT with `repo` + `read:org` scopes (fine-grained tokens may miss org repos)
- **REST endpoints**: `/user`, `/user/repos`, `/user/orgs`, `/orgs/{org}/repos`, `/repos/{owner}/{repo}/pulls`, `/pulls/{n}/reviews`, `/pulls/{n}/merge`, `/commits/{sha}/status`, `/commits/{sha}/check-runs`, `/deployments`, `/deployments/{id}/statuses`
- **GraphQL**: `reviewThreads.isResolved` for accurate unresolved comment counts; `additions`/`deletions` for diff stats (zero extra API calls)
- **Org repos**: Fetches via `/user/orgs` then `/orgs/{org}/repos?type=member` + `type=all`
- **SSO**: Token must be authorized for SSO-enabled orgs after creation

### GitLab

- **Auth**: PAT with `api` + `read_user` scopes (classic token; setup link pre-fills scopes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deployhq/pr-radar](https://github.com/deployhq/pr-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
