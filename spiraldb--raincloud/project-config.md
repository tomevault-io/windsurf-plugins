---
trigger: always_on
description: Guidance for AI coding agents working in this repo. The layout is deliberate and the
---

# AGENTS.md

Guidance for AI coding agents working in this repo. The layout is deliberate and the
invariants below are easy to break by accident — read them before non-trivial changes.

For what raincloud is and how to use it, see [`README.md`](README.md). For the manifest
schema, [`sources.schema.md`](sources.schema.md). For step-by-step procedures,
[`SKILLS.md`](SKILLS.md). `CLAUDE.md` is a symlink to this file.

## Start here

On a fresh clone `outputs/` is empty. That's expected — artifacts are built, not shipped.

```bash
python -m raincloud.pipeline.status --fast --missing-only   # read-only; seconds
python -m raincloud.pipeline.validate_manifest              # schema + registry cross-checks
pytest                                                    # needs --extra dev --extra all
```

Installs are layered. A bare `uv sync --inexact` gets only the loader; builds need
`uv sync --extra build --inexact`, plus the extra a dataset needs for its format or
source: `osm`, `sas`, `excel`, `archives`, `generated` (the TPC-H/TPC-DS generators),
`kaggle`, `huggingface`. `--extra all` installs everything. **Always pass
`--inexact`**: without it, syncing one extra silently uninstalls the others, and a
later Kaggle/HF build fails.

Query the catalog rather than grepping `sources.json` or scrolling
`docs/v2/datasets.md`:

```bash
python -m raincloud.pipeline.list_datasets --handler uci_default --count
python -m raincloud.pipeline.list_datasets --kaggle-tos         # gated behind a one-time click-through (Kaggle or Hugging Face)
python -m raincloud.pipeline.list_datasets --grep '\bgeo' --long
raincloud describe <slug>                                      # one dataset's columns and types, from the catalog
python -m raincloud.pipeline.list_datasets --columns --column-grep PATTERN   # columns across locally built files
python -m raincloud.pipeline.list_datasets --coverage --source parquet       # type coverage of locally built files
python -m raincloud.pipeline.list_datasets --stale-version      # slugs BUILT under an OLDER schema_version (never-built excluded)
```

`--grep` is a regex over `slug short_name full_name description` joined by spaces,
so anchor one slug as `'^<slug> '`; `'^<slug>\b'` also matches `<slug>-hydrated`
and other hyphenated siblings.

Filters AND together across `--handler`, `--license`, `--fetch-type`, `--reader`,
`--vortex`/`--no-vortex`, `--kaggle-tos`, `--stale-version`, `--local`, `--grep`. Output:
default one bare slug per line (a terminal also marks hydrated ones `[hydrated]`),
`--long`, `--json`, `--count`. In `--long`, `recorded` is what the tracked catalog
records (in `--json`, `built_version` / `stale_version`) and `local` is the formats
prepared on this install's disk.

`raincloud.pipeline.browse` is a human-facing TUI — it will hang waiting for keystrokes.
Don't run it from an agent context; point the user at it instead.

## Invariants

1. **`sources.json` is authoritative.** Every row of every derived artifact maps back to
   a spec here. Never hand-edit `docs/*.md` or drop a file into `outputs/` — fix the
   manifest, rebuild, regenerate docs.
2. **`outputs/raw_downloads/<slug>/` is unversioned; `outputs/v{n}/<slug>/<format>/` is
   version-scoped.** Raw upstream bytes don't depend on schema version, so they're cached
   outside the version prefix and shared across versions. Path helpers live in
   `raincloud/pipeline/spec.py` — use them rather than composing paths by hand.
3. **`<scratch_dir>/.recipes/<recipe-hash>/<slug>/` is scratch.** Handlers clean up after
   themselves; `build.py --clean-workdir` removes only the selected generation. Clearing
   it forces re-extraction without disturbing another recipe.
4. **Raincloud code, tests and examples open DuckDB through `raincloud.duckdb_connect`**,
   never `duckdb.connect` directly. It applies the `RAINCLOUD_DUCKDB_*` resource limits
   and `storage_compatibility_version=v1.5.0`, which persistent VARIANT writes require.
5. **`docs/` is split.** Top-level `docs/*.md` is gitignored scratch. `docs/v{n}/*` is the
   tracked canonical set. `raincloud.pipeline.docs` writes to the top level; promoting to
   `docs/v{n}/` is a deliberate manual copy.
6. **A superseded version is frozen.** Artifacts under an older `outputs/v{n}/` are not
   rebuilt and must not be wiped — nothing regenerates them.
7. **`.archive/` is local-only and gitignored.** A fresh clone won't have it. Where other
   docs name it as a fallback, git history is the only one you can rely on.

## How a build works

Orchestrated by `raincloud.pipeline.build`. The pipeline is **canonical-Arrow-spined**:
transform produces Arrow, `write_canonical` persists the one canonical artifact, and
every output format is derived from it by an exporter.

| stage | module | reads | writes |
|---|---|---|---|
| fetch | `fetch.py` | `fetch.*` | `outputs/raw_downloads/<slug>/` |
| extract | `extract.py` | `extract.*` | `<scratch_dir>/.recipes/<hash>/<slug>/` |
| parse | `parse.py` | `parse.*` | in-memory `(Path, Table)` |
| transform | `transform.py` | `transform.*` | in-memory `(slug, Table)` |
| write_canonical | `canonical.py` | transform output | `outputs/v{n}/<slug>/arrow/<slug>.arrow.zstd` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spiraldb/raincloud](https://github.com/spiraldb/raincloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
