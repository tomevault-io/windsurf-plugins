---
trigger: always_on
description: Generated: 2026-04-17
---

# PROJECT KNOWLEDGE BASE

Generated: 2026-04-17
Commit: 1f6da97
Branch: main
Package version: 1.2.7

## OVERVIEW
Codex plugin: intercepts OpenAI SDK calls, routes through ChatGPT Codex backend with multi-account OAuth rotation. Includes CLI management dashboard, per-project/worktree account storage, and deterministic repo hygiene tooling.

## STRUCTURE
```
./
├── index.ts              # plugin entry (7-step fetch pipeline, tool registration)
├── lib/                  # core logic (see lib/AGENTS.md)
│   ├── auth/             # OAuth flow, PKCE, callback server
│   ├── accounts/         # rate limit tracking per account
│   ├── codex-cli/        # CLI state, sync, observability
│   ├── codex-manager/    # settings-hub split into 5 focused sub-modules (all <500 LOC)
│   ├── prompts/          # model-family prompts, GitHub ETag cache
│   ├── recovery/         # conversation state persistence
│   ├── request/          # request transform, SSE, rate-limit backoff
│   ├── storage/          # project root detection, worktree resolution, migrations
│   ├── tools/            # hashline tool helpers
│   └── ui/               # TUI: ansi, auth-menu, theme, select, copy
├── test/                 # vitest suites (see test/AGENTS.md)
│   ├── chaos/            # fault injection tests
│   ├── fixtures/         # test data (v3-storage.json)
│   └── property/         # fast-check property-based tests
├── scripts/              # install, build, hygiene, benchmarks
├── config/               # Codex.json examples (legacy/modern)
├── docs/                 # user + maintainer documentation
│   ├── development/      # architecture, config fields/flow, testing, TUI parity
│   ├── reference/        # commands, settings, storage paths
│   ├── releases/         # v0.1.0, v0.1.1, beta, legacy history
│   └── benchmarks/       # edit format benchmark
├── vendor/               # vendored codex-ai-plugin + codex-ai-sdk dist shims
├── assets/               # static assets
├── .github/              # CI workflows, issue/PR templates, dependabot
└── dist/                 # build output (generated, do not edit)
```

## WHERE TO LOOK
| Task | Location | Notes |
| --- | --- | --- |
| Plugin orchestration | `index.ts` | 7-step request pipeline, tool registration |
| OAuth flow + PKCE | `lib/auth/auth.ts` | token refresh, JWT decode, `REDIRECT_URI` = `127.0.0.1:1455` |
| OAuth callback server | `lib/auth/server.ts` | binds port 1455 |
| Multi-account rotation | `lib/accounts.ts` | health scoring, cooldown, selection |
| Account storage | `lib/storage.ts` | V3 format, per-project/global paths, worktree migration, case-insensitive email dedup |
| Worktree resolution | `lib/storage/paths.ts` | `resolveProjectStorageIdentityRoot`, linked worktree detection, commondir/gitdir validation |
| Storage migrations | `lib/storage/migrations.ts` | V1/V2 → V3 upgrade |
| Request transformation | `lib/request/request-transformer.ts` | model normalization, prompt injection |
| Headers + rate limits | `lib/request/fetch-helpers.ts` | Codex headers, error mapping |
| SSE to JSON | `lib/request/response-handler.ts` | stream parsing |
| Stream failover | `lib/request/stream-failover.ts` | SSE stream recovery |
| Failure policy | `lib/request/failure-policy.ts` | retry/failover decisions |
| Rate limit backoff | `lib/request/rate-limit-backoff.ts` | exponential + jitter |
| Model map | `lib/request/helpers/model-map.ts` | model name normalization |
| Prompt templates | `lib/prompts/codex.ts` | model-family detection, GitHub ETag cache |
| Config parsing | `lib/config.ts` | CODEX_MODE, plugin options |
| CLI manager | `lib/codex-manager.ts` | command dispatcher, `codex auth ...` family |
| Settings hub | `lib/codex-manager/settings-hub/` | interactive settings split into shared/dashboard/backend/experimental/index (all <500 LOC), Q = cancel without save; `settings-hub.ts` is a back-compat re-export stub |
| CLI state/sync | `lib/codex-cli/` | state management, observability, sync, writer |
| Session recovery | `lib/recovery/` | conversation state persistence |
| Graceful shutdown | `lib/shutdown.ts` | cleanup on process exit |
| Health monitoring | `lib/health.ts` | account health status |
| Circuit breaker | `lib/circuit-breaker.ts` | failure isolation |
| Unified settings | `lib/unified-settings.ts` | settings persistence with EBUSY retry |
| Dashboard settings | `lib/dashboard-settings.ts` | dashboard configuration |
| UI layer | `lib/ui/` | ansi, auth-menu, confirm, copy, format, runtime, select, theme |
| Repo hygiene | `scripts/repo-hygiene.js` | `clean --mode aggressive`, `check`, Windows `removeWithRetry` |
| Codex bin wrapper | `scripts/codex.js` | lazy-load auth runtime, graceful missing-dist handling |
| Tests | `test/` | vitest globals, 80% coverage threshold, 225 files, 3418 tests |

## CONVENTIONS
- Source: root `index.ts` + `lib/`; `dist/` is generated output.
- ESLint flat config: `no-explicit-any` enforced, unused args prefixed `_`.
- Tests relax lint rules (see `eslint.config.js`).
- Build copies `lib/oauth-success.html` to `dist/lib/` via `scripts/copy-oauth-success.js`.
- ESM only (`"type": "module"`), Node >= 18.
- Windows filesystem safety: all `fs.rm` calls in scripts/tests use `removeWithRetry` with EBUSY/EPERM/ENOTEMPTY backoff.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndycode/codex-multi-auth](https://github.com/ndycode/codex-multi-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
