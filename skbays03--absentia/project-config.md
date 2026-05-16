---
trigger: always_on
description: A code-hygiene tool that mines patterns a codebase already follows and surfaces
---

# CLAUDE.md — guidance for AI assistants in this repo

## What absentia is

A code-hygiene tool that mines patterns a codebase already follows and surfaces
places that don't follow them. **No LLM in the engine** — purely classical
(tree-sitter + frequent itemset mining + statistics). User-facing tagline:
*"Find the holes your code already drew."*

See `README.md` for the public-facing pitch.

## Architecture in one screen

Four-layer storage model:

```
User layer       suppressions, annotations, config       sticky across runs
Pattern layer    groups, rules, gaps                     derived per run
Entity layer     entities, features, relations           incremental on file change
Raw layer        files, content hashes, parse cache      incremental on disk change
```

Two consumers of the engine library:

- **TUI** — `absentia` (default invocation), Textual-based, primary UX
- **Batch CLI** — `absentia check` for CI, scripts, editor integrations

A future third consumer: a Dev-Dashboard panel that imports the engine as a
Python library or shells out to `absentia check --json`.

## Locked-in decisions

These were debated and chosen deliberately. Don't reverse them silently — if
a reversal seems warranted, surface the reasoning explicitly.

1. **No LLM in the engine.** Determinism, free explanations as a byproduct of
   rule mining, sub-second feedback, and differentiation from saturated AI
   tooling. Embeddings are reserved for an eventual personal-knowledge variant
   if it materializes; LLM only as an optional natural-language query shell,
   never the core.
2. **TUI is the primary UX.** Built with Textual. Batch CLI is the secondary
   scriptable mode. Number keys for view switching; lowercase for actions.
3. **Standalone repo, not a host-app panel.** Designed for any host
   (a developer dashboard, an editor extension, a CI bot) to embed
   via library import or `--json` shellout. The engine has zero
   coupling to the host.
4. **Python + SQLite for the MVP.** Rust + alternate stores reserved for the
   enterprise tier. Don't pre-build that infrastructure.
5. **Stable IDs everywhere.** Entities, rules, gaps, and groups all have
   deterministic IDs derived from their identity (not sequence numbers), so
   suppressions persist across rebuilds.
6. **Architectural seams designed for worst-case; implementation built for
   current case.** Storage interface, extractor plugin shape, group selector
   polymorphism, content-hash-driven incremental — all baked in from day 1.
   Rust port, columnar store, parallelism, etc. are deferred until needed.

## Repo layout

```
src/absentia/        Engine package
  cli.py           Argparse dispatch + subcommand bodies
  tui/             Textual TUI (interactive front-end)
  extractors/      Per-language tree-sitter extractors (17)
  selectors.py     directory / decorator / parent_class group builders
  mining.py        Frequency mining over (group, feature) pairs
  symmetry.py      Symmetry-pair + call-pair detection
  series.py        Numeric-series-gap detection
  enrichment.py    Corpus-level features (sibling_test)
  storage.py       SQLite-backed entity + suppression store
  parallel.py      ProcessPool helpers, default_jobs() policy
  estimator.py     Cold-scan cost model + `absentia est` renderer
  calibration.py   First-run calibration + ~/.absentia/calibration.json
  runs_log.py      Machine-wide ~/.absentia/runs.jsonl accumulator
  settings.py      ~/.absentia/settings.json (jobs_default, etc.)
  progress.py      ProgressBar / Spinner / StepIndicator
  output.py        Gap rendering (text + JSON)
  _color.py        ANSI escape constants for in-place progress UI
  _console.py      rich Console proxy (pytest-capsys-compatible)
tests/             Unit + integration tests
scripts/           Maintenance + diagnostic scripts
  update_ts.py     Tree-sitter grammar version sweep
  scan_remote.py   Sanity-check against public corpora
  diagnose_scan.py Per-stage scan timing for cross-machine debugging
  profile_scan.py  cProfile harness for opt-pass hotspot validation
  release.sh       Interactive bump + tag + push (triggers release-checks.yml)
  local_ci.sh      Mirror of CI checks (ruff + mypy + pytest+cov + mkdocs --strict)
docs/              Mkdocs-material site (Diátaxis structure)
  tutorial/        Learn-by-doing
  how-to/          Task-oriented recipes
  reference/       Look-up authoritative
  explanation/     Concepts + ADRs (in decisions/)
pyproject.toml     Package config
absentia.toml.example   Sample per-project config
README.md          Public-facing pitch
DEFERRALS.md       Publication-blocking items intentionally deferred
CHANGELOG.md       Per-release notes
```

In **user projects** (not this repo):
- `absentia.toml` — committed per-project config + project-wide suppressions
- `.absentia/` — gitignored runtime state directory (entity DB, parse cache, etc.)

## Dev scripts

`scripts/update_ts.py` discovers every installed `tree-sitter*` package and
checks pip for updates. Run periodically as new absentia extractors are added —
keeps grammars current without hardcoding the list.

Three modes:

- **Interactive** (default, when run from a TTY): numbered list + menu.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skbays03/absentia](https://github.com/skbays03/absentia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
