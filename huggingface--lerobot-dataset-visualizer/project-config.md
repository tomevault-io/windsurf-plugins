---
trigger: always_on
description: Always use **bun** (`bun install`, `bun dev`, `bun run build`, `bun test`). Never use npm or yarn.
---

# CLAUDE.md — LeRobot Dataset Visualizer

## Package manager

Always use **bun** (`bun install`, `bun dev`, `bun run build`, `bun test`). Never use npm or yarn.

## Post-process — run after every code change

After making any code changes, always run these commands in order and fix any errors before finishing:

```
bun run format        # auto-fix formatting (prettier)
bun run type-check    # TypeScript: app + test files
bun run lint          # ESLint (next lint)
bun test              # unit tests
```

Or run them all at once (format first, then the full validate suite):

```
bun run format && bun run validate
```

`bun run validate` runs: type-check → lint → format:check → test

## Key scripts

```
bun dev              # Next.js dev server
bun test             # Run all unit tests (bun:test)
bun run type-check   # tsc --noEmit (app) + tsc -p tsconfig.test.json --noEmit (tests)
bun run lint         # next lint
bun run validate     # type-check + lint + format:check
```

## Architecture

### Dataset version support

Three versions are supported. Version is detected from `meta/info.json` → `codebase_version`.

| Version  | Path pattern                                                      | Episode metadata                           | Video                                          |
| -------- | ----------------------------------------------------------------- | ------------------------------------------ | ---------------------------------------------- |
| **v2.0** | `data/{episode_chunk:03d}/episode_{episode_index:06d}.parquet`    | None (computed from `chunks_size`)         | Full file per episode                          |
| **v2.1** | Same as v2.0                                                      | None                                       | Full file per episode                          |
| **v3.0** | `data/chunk-{N:03d}/file-{N:03d}.parquet` (via `buildV3DataPath`) | `meta/episodes/chunk-{N}/file-{N}.parquet` | Segmented (timestamps per episode, per camera) |

### Routing to parsers

`src/app/[org]/[dataset]/[episode]/fetch-data.ts` → `getEpisodeData()` dispatches to:

- `getEpisodeDataV2()` for v2.0 and v2.1
- `getEpisodeDataV3()` for v3.0

### v3.0 specifics

- Episode metadata row has named keys (`episode_index`, `data/chunk_index`, `data/file_index`, `dataset_from_index`, `dataset_to_index`, `videos/{key}/chunk_index`, etc.)
- Integer columns from parquet come out as **BigInt** — always use `bigIntToNumber()` from `src/utils/typeGuards.ts`
- Row-range selection: `dataset_from_index` / `dataset_to_index` allow reading only the episode's rows from a shared parquet file
- Fallback format uses numeric keys `"0"`.."9"` when column names are unavailable
- Episode metadata can span **multiple chunks** (when episode count exceeds `chunks_size`). Always walk via the `iterateEpisodeMetadataFilesV3(repoId, version)` async generator in `fetch-data.ts` — it advances chunk-000 → chunk-001 → … and stops on the first missing `file-000`. Never hardcode `chunk-000`.
- Multi-task episodes: episode-metadata rows carry a `tasks` field (`list[str]`) — prefer it over the legacy single `task_index` lookup. `EpisodeMetadataV3.tasks?: string[]` exposes it.
- `meta/tasks.parquet` lookup: rows are **not** ordered by `task_index`, and the task string lives in a named pandas index (`__index_level_0__`). Always filter by the `task_index` **column** (`row.task_index === taskIndexNum`), never by row position.

### v2.x path construction

```ts
formatStringWithVars(info.data_path, {
  episode_chunk: Math.floor(episodeId / chunkSize)
    .toString()
    .padStart(3, "0"),
  episode_index: episodeId.toString().padStart(6, "0"),
});
// → "data/000/episode_000042.parquet"
```

`formatStringWithVars` strips `:03d` format specifiers — padding must be done by the caller.

## Key files

| File                                              | Purpose                                                                                                                                  |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `src/app/[org]/[dataset]/[episode]/fetch-data.ts` | Main data-loading entry point; v2/v3 parsers; `computeColumnMinMax`                                                                      |
| `src/utils/versionUtils.ts`                       | `getDatasetInfo`, `getDatasetVersionAndInfo`, `buildVersionedUrl`                                                                        |
| `src/utils/stringFormatting.ts`                   | `buildV3DataPath`, `buildV3VideoPath`, `buildV3EpisodesMetadataPath`, padding helpers                                                    |
| `src/utils/parquetUtils.ts`                       | `fetchParquetFile`, `readParquetAsObjects`, `formatStringWithVars`                                                                       |
| `src/utils/dataProcessing.ts`                     | Chart grouping pipeline: `buildSuffixGroupsMap` → `computeGroupStats` → `groupByScale` → `flattenScaleGroups` → `processChartDataGroups` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/lerobot-dataset-visualizer](https://github.com/huggingface/lerobot-dataset-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
