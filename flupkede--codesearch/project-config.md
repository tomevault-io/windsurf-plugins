---
trigger: always_on
description: Add symbol-aware reference lookups to codesearch via `find_impact` MCP tool. Returns file/line-precise references so agents can plan refactors with IDE-level accuracy. MVP is **C# only**; architecture is language-agnostic through per-language `SymbolIndexer` adapters.
---

# AGENTS.md — features/symbol-references

## Goal

Add symbol-aware reference lookups to codesearch via `find_impact` MCP tool. Returns file/line-precise references so agents can plan refactors with IDE-level accuracy. MVP is **C# only**; architecture is language-agnostic through per-language `SymbolIndexer` adapters.

## Implemented Features

- **`find_impact` MCP tool** — returns transitive call-sites for a symbol (name-based or position-based), C# via `scip-csharp` helper
- **`scip-csharp` helper** — .NET 10 CLI wrapping Roslyn. **Two subcommands**:
  - `index` — compile solution, emit **definitions only** (no FindReferencesAsync at rebuild time = 10–50× faster)
  - `find-refs --symbol <key>` — resolve references for ONE symbol on demand (lazy, result cached in `scip_ref_cache`)
- **Opt 1 — external-type filter** — `CollectTypeSymbols` skips all types with no `IsInSource` location (framework/NuGet), 10-100× fewer symbols on large solutions
- **Opt 2 — lazy reference resolution** — rebuild stores definitions only; `find_references()` checks `scip_ref_cache` first, calls `scip-csharp find-refs` on cache miss, then caches result; `block_in_place` in MCP handler for blocking subprocess
- **Opt 3 — incremental merge** — `RebuildScope::Files`: uses position index as reverse map to collect stale symbol keys, merges new definitions (partial-class safe: keeps defs from non-affected files), rebuilds `simple_names` from all current symbols
- **O(1) position lookup** — `scip_positions` LMDB table maps `(file:line)` → `[symbol_keys]`
- **O(1) fuzzy lookup** — `scip_simple_names` LMDB table maps last-segment identifier → `[full_keys]`
- **`scip_ref_cache` LMDB table** — key: SCIP symbol key; value: bincode(Vec<StoredReference>); populated on first `find_impact` per symbol, cleared on any rebuild
- **Bincode schema versioning** — version byte prefix on all LMDB payloads, clear error on mismatch
- **JSON version validation** — rejects scip-csharp index versions other than `"1.0"`
- **Backward compat** — old LMDB indexes (pre-Opt2, with references in `scip_symbols`) still work; `has_legacy_refs` check bypasses lazy invocation
- **Helper failure cache** — `detect_helper()` caches both found and not-found results (`Mutex<Option<Option<PathBuf>>>`)
- **Shared `SymbolIndexerRegistry`** — `ServeState`, `CodesearchService`, and `IndexManager` each own one `Arc<Registry>`; no per-request instantiation
- **`.cs` watcher debounce** — 60s quiet period triggers automatic symbol rebuild
- **`-with-csharp` release variants** — 6 release archives (3 plain + 3 with self-contained helper)
- **Gated integration test** — `csharp_helper_integration` cargo feature for full-pipeline testing
- **CI** — separate `csharp-integration-tests` job in `.github/workflows/ci.yml`
- **Sequential phase-2 startup** — Phase 1 warms repos sequentially, Phase 2 runs gated C# SCIP rebuilds ordered by `last_changed_unix` under `Semaphore(concurrency)` via `CSHARP_SCIP_CONCURRENCY` env (default **2**, clamp [1,4])
- **`repos_meta` tracking** — `RepoMeta` (last_changed_unix, last_scip_indexed_unix) persisted in `repos.json` with debounced save (10s window)
- **TUI C# indicator** — in status column: green `C#·` ready, yellow `C#…` indexing, red `C#!` error; footer shows helper availability; Calls column with tool call count
- **Selective ref cache invalidation** — incremental rebuilds only purge cached refs for affected symbols, not entire cache
- **Phase 3 pre-warm** — after Phase 2 definitions, `scip-csharp batch-find-refs` resolves all uncached symbols in a single workspace session; controlled by `CSHARP_PREWARM_ENABLED` env (default: true)
- **`index symbol` CLI** — `codesearch index symbol [-f] <alias>` for symbol-only rebuild; `--symbols` flag on `index -f` for combined text+symbol rebuild
- **Watcher .csproj grouping** — changed .cs files grouped by .csproj, incremental rebuild per project instead of full solution

## Architecture

### Per-language adapter pattern

`src/symbols/` hosts the adapter layer:

- `mod.rs` — `SymbolIndexer` trait + `SymbolIndexerRegistry` dispatch
- `csharp.rs` — C# adapter (rebuild, find_references, find_references_by_position)
- `scip_parse.rs` — JSON parser for scip-csharp output

### LMDB tables

| Table | Key | Value |
|---|---|---|
| `scip_symbols` | full SCIP key | `[v1, bincode(Vec<StoredReference>)]` — **definitions only** after Opt 2 |
| `scip_positions` | `<file>:<line>` (forward-slash) | `[v1, bincode(Vec<String>)]` |
| `scip_simple_names` | last segment of canonical symbol | `[v1, bincode(Vec<String>)]` |
| `scip_ref_cache` | full SCIP key | `[v1, bincode(Vec<StoredReference>)]` — lazy-resolved references |
| `scip_meta` | `last_rebuild_ts`, `symbol_count` | `Str` |

### Helper detection lookup order

1. `CODESEARCH_SCIP_CSHARP` env var
2. `<codesearch-exe-dir>/helpers/csharp/scip-csharp[.exe]`
3. `$PATH`

Missing helper disables `find_impact` for C# only — all other features keep working.

### Startup phases

| Phase | What | Trigger |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flupkede/codesearch](https://github.com/flupkede/codesearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
