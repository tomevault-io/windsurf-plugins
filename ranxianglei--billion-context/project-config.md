---
trigger: always_on
description: > **This document is the highest-priority specification. All developers (including AI Agents) MUST comply.**
---

# billion-context Development Specification

> **This document is the highest-priority specification. All developers (including AI Agents) MUST comply.**

## 1. Project Overview

**billion-context** is an npm package (`bili` CLI) — a context-compression proxy for AI agents. It sits between an agent client and an upstream LLM provider, injecting acp-kernel's compression pipeline to manage context growth.

### Tech Stack

| Category | Technology |
|----------|-----------|
| Language | TypeScript (strict, ESM) |
| Build | tsup (bundling, inlines acp-kernel) |
| Test | Node.js built-in: `node --import tsx --test tests/*.test.ts` |
| Runtime Dep | `acp-kernel` (bundled at build time, zero runtime deps in dist) |

### Repository Info

| Field | Value |
|-------|-------|
| npm package | `billion-context` |
| CLI | `bili` / `bili-proxy` |
| GitHub | https://github.com/ranxianglei/billion-context |
| License | MIT |

## 2. Architecture

### Module Map

```
billion-context/
├── src/
│   ├── index.ts                  # Entry: runs cli.ts main()
│   ├── cli.ts                    # CLI dispatcher: start/update/version/help
│   ├── server.ts                 # HTTP proxy server, request pipeline
│   ├── config.ts                 # Config loading (file + env + CLI flags)
│   ├── logger.ts                 # Tee logger: file (~/.local/state/) + stderr
│   ├── paths.ts                  # XDG paths (config/cache/state dirs)
│   ├── session.ts                # Session model + in-memory store
│   ├── session-id.ts             # Session ID generation
│   ├── message-id.ts             # Message ref ID generation
│   ├── persist.ts                # On-disk session persistence
│   ├── update.ts                 # Auto-update: checks npm, auto-installs latest
│   ├── stream.ts                 # SSE stream utilities + tag patching
│   ├── stream-openai.ts          # OpenAI-format stream processing
│   ├── stream-responses.ts       # Responses-API stream processing
│   ├── stream-error.ts           # Stream error handling
│   ├── sse-util.ts               # SSE parsing helpers
│   ├── compress-loop.ts          # Compress loop (OpenAI chat format)
│   ├── compress-loop-responses.ts # Compress loop (Responses API format)
│   ├── compress-tool.ts          # compress tool parsing
│   ├── decompress-shared.ts      # Shared decompress logic
│   ├── orphan-gc.ts              # Orphaned block cleanup
│   ├── anthropic.ts              # Anthropic adapter helpers
│   ├── openai.ts                 # OpenAI adapter helpers
│   ├── responses.ts              # Responses API helpers
│   ├── fetch-util.ts             # HTTP fetch with timeout
│   └── util.ts                   # Misc utilities
├── tests/                        # 16 test files, 141 tests
├── tsup.config.ts
└── package.json
```

### Key Design Decisions

1. **acp-kernel is bundled inline** — tsup does NOT list it in `external`, so `dist/index.js` is self-contained (zero runtime deps)
2. **Tags use XML format** `<acp tokens="2" type="text">m00001</acp>` — written with hex escapes (`\x3c`, `\x3e`) to avoid Write/Edit tool stripping
3. **Auto-update**: checks npm registry every 3 min (`CHECK_INTERVAL_MS = 3*60*1000`), first check per process ignores throttle
4. **Tee logger**: all proxy logs go through `src/logger.ts` (file + stderr). Do NOT use `console.error` in server-side modules — use `loggerLog()`.
5. **acp-kernel MUST be pinned to an exact version** (e.g. `"acp-kernel": "0.0.17"`, NEVER `"^0.0.17"`). Because acp-kernel is a build-time dependency that tsup bundles inline into `dist`, a caret range makes the resolved version drift if `package-lock.json` is regenerated or absent, breaking reproducible builds. When bumping acp-kernel: set the exact version in `package.json`, run `npm install` to refresh the lockfile, then rebuild. The `package-lock.json` is committed and kept in sync.

## 3. Development Standards

### Build Commands

```bash
npm run build          # tsup bundle (inlines acp-kernel)
npm run typecheck      # tsc --noEmit --project tsconfig.build.json
npm test               # node --import tsx --test tests/*.test.ts
```

### Local Testing

```bash
npm run build
npm install -g billion-context@latest   # install from registry
bili start --port 8787
```

`npm install -g .` also works (installs from the local directory) and does NOT
create a symlink here — npm copies the package into the global `node_modules`
because `package.json` has proper `bin` + `files` fields. Either approach is
fine; the registry install is preferred for testing the real published
artifact.

### Code Quality

- **No `as any`**, **No `@ts-ignore`**
- **No comments unless absolutely necessary**
- Hex escapes required for any `<acp>` XML in source files
- **No `console.error` in server-side modules** — use `loggerLog(level, msg)` from `src/logger.ts`. The only exceptions are `src/cli.ts` (user-facing CLI errors) and `src/index.ts` (pre-logger startup crash).

## 4. Git Safety Rules (MANDATORY)

| Rule | Enforcement |
|------|-------------|
| **NEVER force-push to `master`** | Under no circumstances. (GitHub branch protection also blocks this.) |
| **NEVER merge PRs** | PR merges are human-only. The Agent MUST NEVER merge. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ranxianglei/billion-context](https://github.com/ranxianglei/billion-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
