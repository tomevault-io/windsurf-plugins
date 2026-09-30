---
trigger: always_on
description: Always use **bun** (`bun install`, `bun dev`, `bun run build`, `bun test`). Never use npm or yarn.
---

# AGENTS.md — LeRobot Dataset Visualizer

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
bun run validate     # type-check + lint + format:check + test
```

## Architecture

### Dataset version support

Three versions are supported. Version is detected from `meta/info.json` → `codebase_version`.

| Version  | Path pattern                                                   | Episode metadata                           | Video                                          |
| -------- | -------------------------------------------------------------- | ------------------------------------------ | ---------------------------------------------- |
| **v2.0** | `data/{episode_chunk:03d}/episode_{episode_index:06d}.parquet` | None (computed from `chunks_size`)         | Full file per episode                          |
| **v2.1** | Same as v2.0                                                   | None                                       | Full file per episode                          |
| **v3.0** | `data/chunk-{N:03d}/file-{N:03d}.parquet`                      | `meta/episodes/chunk-{N}/file-{N}.parquet` | Segmented (timestamps per episode, per camera) |

### Routing to parsers

`src/app/[org]/[dataset]/[episode]/fetch-data.ts` → `getEpisodeData()` dispatches to:

- `getEpisodeDataV2()` for v2.0 and v2.1
- `getEpisodeDataV3()` for v3.0

### `@huggingface/lerobot`

Locating an episode (its video URLs and offsets, its data file and row range) and reading camera sizes go through [`@huggingface/lerobot`](https://github.com/huggingface/huggingface.js/tree/main/packages/lerobot). Don't build v3 data, video or episode-metadata paths by hand.

- Create datasets with `leRobotDataset(repoId)` in `fetch-data.ts`: it passes the `DATASET_URL` endpoint and `authHeaders()`.
- `episodes({ offset, limit })` reads the v3 episode index across every metadata chunk. `offset` is a position in the listing, not an `episode_index`.
- `episode.data.fromRow` / `toRow` are row offsets **inside `episode.data.url`**, not dataset-wide frame indexes. Pass them straight to `rowStart` / `rowEnd`; never subtract the file's first `index`. `episode.data` is optional: it is absent when the data file can't be read.
- Episode charts come from `frames()` (numeric and boolean series, bookkeeping columns left out). `loadEpisodeFrames` reads the task and language columns (`TEXT_COLUMNS`) from the same file alongside it, since `frames()` returns numbers only.
- Camera sizes come from `parseInfo(...).cameras`. Never read them from `shape[0]` / `shape[1]`: some datasets declare channel-first shapes (`[3, H, W]`).
- To upgrade, bump the version in `package.json` and run `bun install`. Read the package's changes first: 0.0.4 changed what `fromRow` / `toRow` mean.

### v3.0 specifics

- Integer columns from parquet come out as **BigInt** — always use `bigIntToNumber()` from `src/utils/typeGuards.ts`
- Fallback format uses numeric keys `"0"`.."9"` when column names are unavailable
- Multi-task episodes: episodes carry a `tasks` list (`list[str]`) — prefer it over the legacy single `task_index` lookup. `EpisodeMetadataV3.tasks?: string[]` exposes it.
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/lerobot-dataset-visualizer](https://github.com/huggingface/lerobot-dataset-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
