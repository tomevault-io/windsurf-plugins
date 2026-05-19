---
trigger: always_on
description: **Last reviewed:** 2026-04-27
---

# PROJECT KNOWLEDGE BASE

**Last reviewed:** 2026-04-27
**Branch:** main

> Verify against current HEAD: `git rev-parse HEAD`. Code map line numbers reflect the snapshot above; rerun `grep -n` if they look stale.

## OVERVIEW

Unified TypeScript interface for querying web archive providers (Wayback Machine, Archive.today, Common Crawl, Perma.cc, WebCite). Built on the unjs ecosystem: ofetch, unstorage, c12, consola, ufo, obuild, changelogen.

## STRUCTURE

```
omnichron/
├── src/
│   ├── index.ts          # barrel - public API surface
│   ├── archive.ts        # createArchive factory + combineResults
│   ├── types.ts          # all public interfaces/types
│   ├── _providers.ts     # provider-specific option types (internal)
│   ├── config.ts         # c12-based config loading with caching
│   ├── storage.ts        # unstorage caching layer
│   ├── providers/        # one file per archive source + barrel
│   └── utils/            # parallel processing, response helpers, domain normalization
├── test/                 # mirrors src/ structure, one .test.ts per module
├── playground/           # Nuxt app (Cloudflare preset) for manual provider testing
└── .github/workflows/    # ci.yml + autofix.yml
```

## WHERE TO LOOK

| Task                      | Location                                                | Notes                                                                              |
| ------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Add a provider            | `src/providers/` + register in `src/providers/index.ts` | Copy wayback.ts as template. Default export factory fn returning `ArchiveProvider` |
| Provider-specific options | `src/_providers.ts`                                     | Extend `ArchiveOptions`, add to `ProviderOptions` map                              |
| Change public API         | `src/index.ts`                                          | Barrel re-exports only. Types via `export type *`                                  |
| Modify caching            | `src/storage.ts`                                        | Key format: `{prefix}:{providerSlug}:{domain}:{limit?}`                            |
| Config defaults           | `src/config.ts` → `getDefaultConfig()`                  | c12 loads from `.omnichron`, `omnichron.config.ts`, `package.json`                 |
| Response helpers          | `src/utils/_utils.ts`                                   | `createSuccessResponse`, `createErrorResponse`, `mergeOptions`                     |
| Parallel processing       | `src/utils/_utils.ts` → `processInParallel`             | Concurrency + batch control                                                        |
| CDX row mapping           | `src/utils/_utils.ts` → `mapCdxRows`                    | Wayback/CommonCrawl share CDX format                                               |
| Test a provider           | `test/{provider}.test.ts`                               | Uses vitest, mocks with `vi.fn()`                                                  |
| Manual testing            | `playground/server/api/snapshots/`                      | One Nuxt endpoint per provider                                                     |
| Integration test          | `test.sh`                                               | Builds lib, then builds playground against it                                      |

## CODE MAP

| Symbol                      | Type      | Location              | Role                                                                                        |
| --------------------------- | --------- | --------------------- | ------------------------------------------------------------------------------------------- |
| `createArchive`             | function  | archive.ts:56         | Core factory. Accepts provider(s) + options, returns `ArchiveInterface`.                    |
| `UnsupportedOperationError` | class     | archive.ts:18         | Thrown by `getPages()` when every queried provider is unsupported. Carries `providers` list. |
| `providers`                 | object    | providers/index.ts:14 | Lazy-loading factory. Each method returns `Promise<ArchiveProvider>`.                       |
| `ArchiveInterface`          | interface | types.ts:127          | Public API: `snapshots()`, `getPages()`, `use()`, `useAll()`.                               |
| `ArchiveProvider`           | interface | types.ts:117          | Provider contract: `name`, `slug?`, `snapshots()`.                                          |
| `ArchiveResponse`           | interface | types.ts:100          | `{ success, pages, error?, unsupported?, unsupportedReason?, _meta?, fromCache? }`.         |
| `ArchivedPage`              | interface | types.ts:61           | `{ url, timestamp, snapshot, _meta }`.                                                      |
| `UnsupportedProviderRecord` | interface | types.ts:84           | `{ provider, reason }` row used in `_meta.unsupportedProviders`.                            |
| `OmnichronConfig`           | interface | config.ts:8           | Config shape: `storage` + `performance` + env overrides.                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oritwoen/omnichron](https://github.com/oritwoen/omnichron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
