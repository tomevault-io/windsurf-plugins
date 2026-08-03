---
trigger: always_on
description: MCP server (AGPL-3.0, **Bun + TypeScript + Effect v4**) that indexes Brazilian public-procurement
---

# AGENTS.md — dados-publicos-mcp

MCP server (AGPL-3.0, **Bun + TypeScript + Effect v4**) that indexes Brazilian public-procurement
legislation and related public data **locally** and serves it to MCP clients over stdio. Public data
only — **zero secrets, zero API keys**. Each data source is a self-contained vertical slice over one
shared local database.

## One world — Effect-native

The v1→v2 cutover has landed. **There is no legacy layer anymore.** The whole tree is the
Effect-native v2 world: `better-result`, `evlog`, `zod`, `dayjs`, `bun:sqlite`, `cac`,
`src/modules/**` and `src/core/**` are **deleted**. There is one PGlite database, one tool layer over
the low-level MCP SDK `Server`, and one CLI. Every file you touch follows the v2 idiom below.

| | **v2 — the only world** |
|---|---|
| Lives in | `src/kernel/**`, `src/sources/**`, `src/serve/**`, `src/runtime.ts`, `src/index.ts` |
| Stack | Effect `4.0.0-beta.81` + PGlite + Drizzle + local embeddings |
| Errors | `Schema.TaggedErrorClass` + `Effect.fail` (never `throw`) |
| Identifiers | **English** — pt-BR **only** inside user-facing error/description strings |
| Static rules | universal **+ v2-strict** on `src/kernel/` and `src/sources/` |

## The tree

```
src/
  kernel/              shared infrastructure (the toolbox)
    db/                ONE PGlite db: client.ts, persistence.ts, data-dir.ts, ddl.ts,
                       schema-registry.ts, provision.ts, relations.ts, schemas/*
    embed/             local embeddings (@huggingface/transformers, multilingual-e5-small)
    http/              the gold-standard HTTP client (getJson + Schema decode + classified retry)
    csv/ zip/ xlsx/ text/   parsing kernels reused across sources
  sources/<x>/         21 vertical slices: catalog.ts + indexer.ts + store.ts
  serve/               the MCP tool layer (declared tools over the low-level SDK Server)
  runtime.ts           ManagedRuntime over AppLayer (21 sources ⊕ Infra)
  index.ts             the CLI (effect/unstable/cli + @effect/platform-bun)
```

The 21 sources: `legislacao`, `ibge-localidades`, `cnae`, `catmat-catser`, `sicaf-fornecedores`,
`sancoes-cgu`, `receita-cnpj`, `tse-eleitoral`, `camara-deputados`, `querido-diario`, `capag`, `pncp`,
`tcu-inidoneos`, `ibge-economia`, `senado`, `cmed-anvisa`, `siconfi-fiscal`, `transferegov`,
`painel-precos`, `transparencia-despesas`, `sinapi`. The last 8 (heavy) are CLI-only: `index <fonte>`.

## Quick commands

```bash
bun install                                   # Bun only — no npm/yarn/pnpm

bun run start                                 # serve over stdio  (= bun src/index.ts, no subcommand)
bun run index                                 # index all LIGHT sources (= bun src/index.ts index)
bun src/index.ts index legislacao             # one source
bun src/index.ts index --include-heavy        # include heavy downloads
bun src/index.ts index pncp --mes 2026-01     # scoped index (pncp month window)
bun src/index.ts index querido-diario --ufs SP,RJ --anos 2024,2025

bun run check                                 # THE GATE — make it green before finishing any change
```

`bun run check` = `typecheck` (`tsc --noEmit`, the type gate — no `dist`, `src` ships as-is)
`+ lint:errors` (the AST checker) `+ test:unit` (`vitest run`). There is **no `bun test`** anymore and
**no CI** — `check` is the only gate; `prepublishOnly` re-runs it. The integration suite is separate
(`bun run test:integration`, `vitest.integration.config.ts`).

**Alchemy owns the local infrastructure.** The user's machine is treated as infra: the dataDir, the
four PGlite extensions, and every table's DDL are provisioned through Effect-native Alchemy resources
in `infra/` (outside the v2-strict tier, but the same Effect idiom — `FileSystem`/`Path`/`Config`, no
node `fs`-sync, no `process.env`). The single source of the schema is `src/kernel/db/schema-registry.ts`
(`dbSources` → `allDbTables` / `tableGroupsBySource` / `schemaDdlText`); `src/kernel/db/provision.ts`
(`provisionSchema`) creates the extensions + every table.

- `bun run infra:deploy` provisions the database (extensions + DDL) via the `Mcp.LocalDatabase`
  resource. Idempotent: a redeploy is a **no-op** while the schema sha256 (`schemaHash()` over
  `schemaDdlText`) is unchanged and the dataDir still exists on disk. `bun run infra:destroy` tears the
  `DadosPublicosLocal` stack down (it does **not** wipe the dataDir).
- `bun run index [<fonte>]` is **Alchemy-backed too**: it deploys `Mcp.LocalIndex` resources
  (`deployIndex`/`deployAll` in `infra/local-index.run.ts`, stack `DadosPublicosIndex`). Each index
  runs `provisionSchema` then the source's index pipeline in the **same** runtime connection, so a
  fresh checkout can `index` directly without a separate `infra:deploy`. A re-index is a no-op unless
  the scope (`scopeHash`) or schema (`schemaHash`) changed.

The runtime never provisions: `serve` only **opens** the already-provisioned dataDir and queries it.

## The static checks (`bun run lint:errors`)

`tooling/static-checks/check-declarative-errors.ts` is **AST-based** (typescript compiler,
`ts.createSourceFile` walk) — not line-based. It dispatches on `ts.SyntaxKind` so it can tell a TS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Licitei/dados-publicos-mcp](https://github.com/Licitei/dados-publicos-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
