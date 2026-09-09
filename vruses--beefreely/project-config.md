---
trigger: always_on
description: Project context for AI agents. Read this first before any development work on this repo.
---

# AGENTS.md

Project context for AI agents. Read this first before any development work on this repo.

## Project: beefreely (哔哩免登录)

A Tampermonkey/Violentmonkey userscript that intercepts Bilibili (bilibili.com) API requests to give non-logged-in users a near-logged-in experience — 1080p video, full comments, subtitles, live danmaku, no login popups.

**Runtime**: Browser userscript, executes at `document-start`.
**GitHub**: https://github.com/vruses/beefreely

## Tech Stack

| Category | Tech |
|---|---|
| Language | TypeScript (strict) |
| Build | Vite + `vite-plugin-monkey 5` |
| Package manager | pnpm |
| Lint/Format | Biome (single quotes, no semicolons, 120 char width) |
| Git hooks | Lefthook (pre-commit: `pnpm lint`) |
| Reactivity | `@vue/reactivity` (loaded via CDN at runtime) |
| Protobuf | `protobufjs` (loaded via CDN at runtime) |
| Hashing | `ts-md5` |
| IndexedDB | `dexie` |
| Utilities | `lodash-es` |
| Types | `type-fest` |
| Versioning | `standard-version` (conventional commits) |

## Build Commands

```bash
pnpm build        # typecheck + build (unminified output → dist/beefreely.user.js)
pnpm build:min    # typecheck + minified build (→ dist/beefreely.min.user.js)
pnpm typecheck    # tsc only
pnpm lint         # biome check
pnpm lint:fix     # biome check --fix
pnpm format       # biome format --write
pnpm dev          # vite dev server
pnpm preview      # vite preview
pnpm release      # standard-version (bump version + changelog)
```

## Project Structure

```
src/
├── main.ts                      # Entry point, imports @/core
├── core/
│   ├── index.ts                 # Main logic: login detection, subdomain routing, hook injection
│   ├── config.ts                # domainConfig: shared hooks + per-subdomain children + blacklist
│   └── lifecycle.ts             # Document readyState lifecycle manager
├── constants/
│   ├── index.ts                 # Re-exports sign.ts (img_key, sub_key)
│   ├── sign.ts                  # WBI image/sub key extraction from localStorage
│   └── utils.ts                 # toResult() wrapper for Bilibili API response envelope
├── store/
│   ├── user.ts                  # Reactive isLogin store (watches for real login → clears all hooks)
│   └── playHistory.ts           # Dexie-backed IndexedDB store for local video watch history
├── types/
│   ├── response.ts              # ResultType<T> generic
│   └── window.d.ts              # Window type augmentations
├── utils/
│   ├── ajax/
│   │   ├── index.ts             # RequestHooker singleton wrapping ajaxHooker
│   │   └── ajax-hooker.d.ts     # Type defs for ajax-hooker library
│   ├── websocket/
│   │   └── intercept.ts         # WebSocket send() monkey-patch for live danmaku
│   ├── wbi-sign.ts              # WBI signature generation (encWbi)
│   ├── web-key.ts               # Key extraction from wbi URLs
│   └── parseParams.ts           # Generic URLSearchParams parser with schema
├── bilibili/                    # Per-page/subdomain hook modules
│   ├── shared/                  # Applied on ALL subdomains
│   │   ├── index.ts             # [useNav, useReply, useReplyShareUrl]
│   │   ├── hooks.ts             # useNav — mock login state; useReply — strip credentials; useReplyShareUrl — fix share URI
│   │   └── model/
│   │       ├── constants.ts     # mockUserInfoResult
│   │       └── types.ts
│   ├── www/                     # www.bilibili.com
│   │   ├── index.ts             # Aggregates history + video + bangumi + opus
│   │   ├── history/             # Local watch history (IndexedDB-backed)
│   │   │   ├── index.ts         # [useHistoryCursor, useHistoryClear, useHistoryDelete, useHistorySearch]
│   │   │   ├── hooks.ts         # Cursor pagination, search, delete, clear
│   │   │   └── model/types.ts   # HistoryRecord, CursorParam, SearchParam types
│   │   ├── video/               # Video player hooks
│   │   │   ├── index.ts         # [usePlayer, usePlayurl, usePlayurl2, useRelation, useArchiveRelation, useDmView(), ...playHistory]
│   │   │   ├── apiConfig.ts     # Endpoint URL constants (placeholder)
│   │   │   ├── hooks/
│   │   │   │   ├── index.ts     # usePlayer, usePlayurl, usePlayurl2, useRelation, useArchiveRelation, useDmView
│   │   │   │   ├── useCrypt.ts  # Subtitle encryption utilities
│   │   │   │   ├── useSubtitle.ts # Subtitle hook factory
│   │   │   │   └── plugin/playHistory/  # Video watch history reporting
│   │   │   │       ├── index.ts         # [useHeartbeat, usePlayHistory, useVideoDetail]
│   │   │   │       ├── useHeartbeat.ts  # Heartbeat reporting
│   │   │   │       ├── usePlayHistory.ts # Play history list reporting
│   │   │   │       └── useVideoDetail.ts # Video detail for history
│   │   │   └── model/
│   │   │       ├── constants.ts    # relationResult, archiveRelationResult
│   │   │       ├── DmWebView.ts    # Protobuf DmWebView definition
│   │   │       └── types.ts        # PlayerUserInfo
│   │   ├── bangumi/               # Bangumi (番剧) hooks
│   │   │   ├── index.ts           # [useBangumiLogin, ...playHistory]
│   │   │   ├── hooks/
│   │   │   │   ├── index.ts       # useBangumiLogin — sync login status for playview
│   │   │   │   └── plugin/playHistory/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vruses/beefreely](https://github.com/vruses/beefreely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
