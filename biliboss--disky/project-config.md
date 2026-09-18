---
trigger: always_on
description: Fast macOS disk analyzer and cleanup CLI in Rust — `ncdu` / `dust` / GrandPerspective alternative.
---

# disky

Fast macOS disk analyzer and cleanup CLI in Rust — `ncdu` / `dust` / GrandPerspective alternative.

## Stack

| Crate | Purpose |
|-------|---------|
| `jwalk 0.8` | Parallel traversal — use `Parallelism::RayonNewPool(cpus)`, NOT `.num_threads()` (wrong API) |
| `duckdb 1.1` (bundled) | Storage — Appender API for batch inserts, 50k/batch |
| `ratatui 0.29` + `crossterm 0.28` | TUI — ncdu-style tree, requires real TTY |
| `flume 0.11` | Bounded channel walker→writer, cap 256 |
| `memchr 2` | `memrchr(b'.', ...)` for ext extraction (2-3x faster than `Path::extension`) |

## Distribution

Five indexable surfaces — `disky schema` + JSON envelopes are the agent contract:

| Surface | URL | Notes |
|---|---|---|
| Repo | `github.com/biliboss/disky` | 18 topics, SEO description, social-preview pending |
| crates.io | `crates.io/crates/disky` | `cargo install disky` (primary install path) |
| docs.rs | `docs.rs/disky` | auto-built on crate publish |
| Releases | `github.com/biliboss/disky/releases` | `aarch64-apple-darwin` + `x86_64-apple-darwin` tarballs per tag |
| GitHub Pages | `biliboss.github.io/disky` | source: `main` `/docs`, `.nojekyll` bypass + `docs/index.html` (brut palette) |

Install order in README is the order to recommend: `cargo install disky` → binary download → `cargo install --git`. Homebrew tap is "coming soon" (earn-back: someone actually asks).

## Snapshot diff

`disky diff <a> <b> [--limit N]` compares two snapshots and reports the
files that grew, shrank, were added, or were removed — ordered by absolute
delta. Both arguments accept `@latest`, an ID, or a path. JSON `records`
hold `DiffRow { path, kind: added|removed|grew|shrank, size_a, size_b, delta }`.

Useful for "what changed since the last cleanup": scan before, scan after,
`disky diff old_id new_id`.

## Cleanup

`disky cleanup` greps the snapshot for known disk-hoggy directories
(`node_modules`, `target`, `__pycache__`, `.next`, `dist`, `build`,
`.venv`/`venv`, `.gradle`, `.pytest_cache`). Default is dry-run.

```
disky cleanup --snapshot @latest                              # preview
disky cleanup --target node_modules,target --apply            # rm -rf
disky cleanup --target node_modules,target --apply --reversible  # → ~/.Trash
```

JSON output: `{kind:"cleanup", applied:bool, removed:[paths], records:[CleanupHit], summary:[CategorySummary], total_bytes:N}`.

**Performance (v0.11.0):** Single grouped range-join with a TEMP table
of target dirs. DuckDB plans zone-maps + IEJoin in one pass.
**79 s → < 1 s on a 1.77 M-row snapshot.** Integration test
`cleanup_is_fast_on_large_snapshot` in `tests/lib_integration.rs`.
Prior algos (per-target loop, LEFT JOIN with LIKE) preserved in
`src/cleanup.rs` history comment.

## `.diskyignore` loader (v0.11.0)

Module `src/ignore.rs` (170 LOC, std-only) provides:

- `default_skip_substrings()` — built-in baseline (node_modules, target, …).
- `load_diskyignore_chain(scan_root)` — walks ancestor dirs up to `$HOME`
  (or `/`), parses gitignore-subset format (one substring per line,
  `#` comments, blank lines skipped, no globs in v1).
- `should_skip(basename, patterns)` — substring match.

Module ships standalone in v0.11.0. Wiring into the live `src/scan.rs`
skip-list is a v0.11.1 deliverable (~10-line edit). 5 unit tests cover
empty dir, single file, comments, ancestor chain, malformed lines.

## N-snapshot growth — OLS (v0.11.0)

```
disky growth --over-n N [--fill-target <bytes>]   # default N=5, min 3
```

Fits an ordinary-least-squares line through `(snapshot_ts, size)` for
each directory present in the latest snapshot across the N most-recent
snapshots. Envelope `kind="growth_n"`:

```
{path, slope_bytes_per_day, r2, projected_fill_date, latest_bytes,
 n_snapshots, sample_paths_ts[[ts,bytes]…]}
```

Sorted by `slope_bytes_per_day DESC`. Pure f64, no stats crate. Math in
`src/query.rs::linfit_slope_r2`. The 2-snapshot `disky growth` path is
untouched — `--over-n` is additive.

## Pattern classifier (v0.11.0)

Module `src/pattern.rs` (325 LOC, std-only) classifies a directory's
size-over-time series into:

| Pattern | Meaning |
|---|---|
| `log_shaped` | Big initial growth that levels off (caches warming) |
| `burst` | Sudden late jump (download dump, import) |
| `stable` | Roughly flat across the series |
| `declining` | Trending down (cleanup landed, log rotation) |
| `unknown` | Mixed / unclear / < 3 samples |

Decision tree thresholds in `src/pattern.rs`:
- `SPIKE_RATIO = 4.0` (max-stride / median-stride for log_shaped/burst)
- `STABLE_RATIO = 3.0` (raised from sub-agent default 1.5 post-merge)
- `DECLINING_SLOPE_FRAC = -0.01`, `DECLINING_R2_MIN = 0.5`

10 unit tests cover each branch + edges. CLI flag `disky churn
--classify` wiring is a v0.11.1 deliverable; the module is callable
in-process today.

## Schema introspection

`disky schema` prints a JSON document describing commands, record shapes,
error codes, and snapshot-ref forms. Pair it with `--format json` on any
command to let an agent bind without prompt-engineering.

## Snapshot references

All query subcommands accept `--snapshot <ref>` where `<ref>` is:

| Form | Example | Resolves to |
|------|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [biliboss/disky](https://github.com/biliboss/disky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
