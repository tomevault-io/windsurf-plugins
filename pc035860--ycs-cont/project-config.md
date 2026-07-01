---
trigger: always_on
description: This file provides practical guidance for contributors working on YCS.
---

# CLAUDE.md

This file provides practical guidance for contributors working on YCS.

## Project Overview

YCS (YouTube Comment Search) is a browser extension for Chrome/Firefox that loads, searches, filters, and exports:
- YouTube comments/replies
- chat replay
- video transcript

Core stack:
- MV3 extension
- TypeScript + Parcel 2
- Node.js 22+

Default development branch: `v2-source`

---

## Working Directories

- Repo root: release scripts, docs, packaging
- `app/`: extension source, tests, build outputs
- `app/src/source/web-resources/`: main UI + search behavior
- `app/src/source/web-resources/features/`: reusable UI behavior modules (Shorts, transcript, recorder, intent state)
- `app/src/source/utils/innertube/`: data loading from YouTube internals
- `app/docs/`: technical docs and behavior specs

---

## Essential Commands

Run from `app/` unless noted.

```bash
# Dev/build
npm run dev
npm run build
npm run rebuild

# Quality
npm run lint
npm run typecheck
npm test
npm run format
npm run format:check

# Clean
npm run rm
```

From repo root:

```bash
make release TYPE=patch
make release TYPE=minor
make release TYPE=major
```

Manual release scripts (root):

```bash
./scripts/bump-version.sh patch
./scripts/build-extension.sh chrome
./scripts/build-extension.sh firefox
./scripts/package-extension.sh chrome
```

---

## Commit/Hook Reality

- Husky pre-commit is configured at `app/.husky/pre-commit`.
- Current pre-commit pipeline runs:
  1. `npm run typecheck`
  2. `npm test`
  3. `npx lint-staged`
- Keep staged changes minimal before committing, because tests run pre-commit.

---

## Architecture Snapshot

Three-layer message flow:

1. Web page layer (`web-resources`)  
2. Content script bridge (`content-scripts`)  
3. Service worker (`background.ts`)

Main state container:
- `app/src/source/web-resources/state.ts`
- Holds loaded datasets, search counters, selected sort settings, and abort controller

Search entry point:
- `app/src/source/web-resources/appController.ts`

Search engines:
- `app/src/source/web-resources/search/commentsSearch.ts`
- `app/src/source/web-resources/search/chatSearch.ts`
- `app/src/source/web-resources/search/transcriptSearch.ts`

Origin chain (parent-comment context for replies):
- `app/src/source/web-resources/ui/originChain.ts` — single source of truth for `expandOriginChainFor` and `autoExpandAllRepliesIn`
- `commentInteractions.ts:handleOpenCommentAll` and `render.ts:renderCommentsResult` are the only callers
- **DOM scope / review note:** Chain wrappers use document-wide checks and removal (`ycs-com-all-*` via `getElementById` / `querySelectorAll` in `originChain.ts` and `commentInteractions.ts`). That matches the current navigation model: only one YCS search result subtree is active at a time (e.g. `executeSearchBasedOnType` runs `timestampViz` by clearing `#ycs-search-result` in `timestampVizHandler.ts` before chart + `#ycs-timestamp-interval-results`, so “main comment results” and “interval results” are not simultaneously present as sibling panels in normal use).
- If a future change introduces **two** persistent comment result roots in the same document with the same reply key, scope origin-chain existence/removal to each result container (or disambiguate ids); treat that as an explicit product/architecture change, not an assumed regression from the current branch.

---

## Philosophy and Conventions

1. Behavior stability over clever refactors  
   If a UX behavior is already established, treat changes as product decisions, not refactor side effects.

2. Regression-first review mindset  
   If behavior changes without explicit requirement, flag it as regression.

3. Explicit user action for search refresh  
   Text input changes alone should not silently alter result lists.

4. Keep search/filter logic centralized  
   Prefer changing behavior in `appController.ts` / search modules, not ad-hoc UI patches.

5. Documentation is part of the feature  
   New recurring behavior rules belong in `app/docs/` so reviewers can enforce them.

6. Write docs/comments in English  
   Keep technical docs concise and state-focused.

7. Shorts integration should be state-driven, not result-driven  
   On Shorts pages, UI visibility decisions should follow user intent state and explicit actions.

8. Keep Shorts layout changes local to YCS  
   Avoid modifying non-YCS YouTube panel sizing unless explicitly required.

---

## High-Value Gotchas (Frequent Regression Sources)

### 1) Search text clear vs filter clear are different operations

- `#ycs_btn_search_clear_text` (input-side clear):
  - Clears text
  - If a filter is active, re-runs search with empty query + same filter
  - If no filter is active, clears rendered results and shows "Search cleared"

- `#ycs_btn_clear` (filter clear):
  - Clears active filter
  - If query exists, re-runs search with query preserved and no filter
  - If query is empty, clears rendered results and shows "Search cleared"

Do not merge these semantics.

### 2) Backspace-to-empty does not auto-search

- Input events only update clear-text button visibility.
- Search refresh happens on explicit actions (Enter/Search/filter clicks).

### 3) Filter model is single-select

- Only one `.ycs_btn_active` at a time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pc035860/YCS-cont](https://github.com/pc035860/YCS-cont) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
