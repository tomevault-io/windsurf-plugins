---
trigger: always_on
description: **Last reviewed:** 2026-09-02
---

# PROJECT KNOWLEDGE BASE

**Last reviewed:** 2026-09-02
**Branch:** main

> Verify against current HEAD: `git rev-parse HEAD`. Code map line numbers reflect the snapshot above; rerun `grep -n` if they look stale.

## OVERVIEW

Unified TypeScript interface for querying web archive providers (Wayback Machine, Arquivo.pt, Webarchiv Österreich, Archive.today, Memento/MemGator, Common Crawl, Perma.cc, WebCite). Built on the unjs ecosystem: ofetch, unstorage, c12, consola, ufo, obuild, changelogen.

## STRUCTURE

```
archives/
├── src/
│   ├── index.ts          # barrel - public API surface
│   ├── archive.ts        # createArchive factory + combineResults/combineContentResults
│   ├── diff.ts           # bounded capture comparisons with checked provenance
│   ├── types.ts          # all public interfaces/types
│   ├── _providers.ts     # provider-specific option types (internal)
│   ├── config.ts         # c12-based config loading with caching
│   ├── storage.ts        # unstorage caching layer
│   ├── tool-operations.ts # executors shared by MCP, Pi and OMP
│   ├── mcp.ts            # createMcpServer() over the shared executors
│   ├── cli.ts            # citty entry (bin: archives), lazy `mcp` subcommand
│   ├── commands/mcp.ts   # `archives mcp` - stdio transport
│   ├── version.ts        # package.json version, single source
│   ├── providers/        # one file per archive source + barrel
│   └── utils/            # _utils.ts: parallel work, response helpers, domain/timestamp
│                         # _content.ts: capture reading, WARC, charset, html-to-text
├── build.config.ts       # obuild: one bundle, four inputs (shared chunks)
├── test/                 # mirrors src/ structure, one .test.ts per module
├── packages/pi/extensions/
│   └── archives.ts       # Pi tool/command surface shipped via package.json pi.extensions
├── packages/omp/extensions/
│   └── archives.ts       # OMP tool/command surface shipped via package.json omp.extensions
├── playground/           # Nuxt app (Cloudflare preset) for manual provider testing
├── docs/                 # Docus site: guide, provider pages, live timeline explorer on Workers
└── .github/workflows/    # ci.yml + autofix.yml
```

## WHERE TO LOOK

| Task                       | Location                                                          | Notes                                                                                             |
| -------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Add a provider             | `src/providers/` + register in `src/providers/index.ts`           | Copy wayback.ts as template. Default export factory fn returning `ArchiveProvider`                |
| Provider-specific options  | `src/_providers.ts`                                               | Extend `ArchiveOptions`, add to `ProviderOptions` map                                             |
| Change public API          | `src/index.ts`                                                    | Barrel re-exports only. Types via `export type *`                                                 |
| Modify caching             | `src/storage.ts`                                                  | Key format: `{prefix}:{providerSlug}:{domain}:{limit?}`                                           |
| Config defaults            | `src/config.ts` → `getDefaultConfig()`                            | c12 loads from `.archives`, `archives.config.ts`, `package.json`                                  |
| Response helpers           | `src/utils/_utils.ts`                                             | `createSuccessResponse`, `createErrorResponse`, `mergeOptions`                                    |
| Read an archived body      | `src/utils/_content.ts`                                           | Capture selection, `id_` playback, WARC ranges, transfer/content encodings, charset, `htmlToText` |
| Compare two captures       | `src/diff.ts` + `src/tool-operations.ts`                          | Pure bounded diff, retrieval from one provider, and paged tool rendering                          |
| Add content to a provider  | provider file → `override content()`                              | Optional on `ArchiveProvider`; a provider that cannot serve bodies says so instead                |
| Parallel processing        | `src/utils/_utils.ts` → `processInParallel`                       | Concurrency + batch control                                                                       |
| CDX row mapping            | `src/utils/_utils.ts` → `mapCdxRows`                              | Wayback/CommonCrawl share CDX format                                                              |
| Test a provider            | `test/{provider}.test.ts`                                         | Uses vitest, mocks with `vi.fn()`                                                                 |
| Manual testing             | `playground/server/api/snapshots/`                                | One Nuxt endpoint per provider                                                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agntn/archives](https://github.com/agntn/archives) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
