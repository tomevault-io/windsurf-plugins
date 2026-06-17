---
trigger: always_on
description: **lightcone-cli** is Lightcone Research's agentic layer for ASTRA (Agentic Schema for Transparent Research Analysis). It ships the `lc` executable and Claude Code skills/hooks used during interactive analysis work.
---

# CLAUDE.md

## Project Overview

**lightcone-cli** is Lightcone Research's agentic layer for ASTRA (Agentic Schema for Transparent Research Analysis). It ships the `lc` executable and Claude Code skills/hooks used during interactive analysis work.

- **ASTRA** = pure specification: schema, validation, prior insights & findings, evidence verification, helpers, minimal CLI
- **lightcone-cli** = agentic layer: Claude Code skills, project scaffolding, **Snakemake-based execution**, container builds

lightcone-cli depends on ASTRA. The `astra` CLI handles spec operations; the `lc` CLI handles execution and agent operations.

### Namespace contract

`lightcone-cli` ships the `lightcone.*` namespace via PEP 420 implicit namespace packages. **`src/lightcone/` must not contain an `__init__.py`** — that would turn the namespace into a regular package and break coexistence with future sibling distributions (`lightcone-ui`, etc.).

Any new `lightcone-*` package must:

1. Use src-layout (`src/lightcone/<name>/…`).
2. Not create `src/lightcone/__init__.py`.
3. Ship only its own subpackage under `src/lightcone/<name>/`.

## Architecture

The execution layer is a thin shim over Snakemake. The integrity layer (per-output content-addressed manifests) is the only thing we own substantively.

```
astra.yaml ── snakefile generator ──> .lightcone/Snakefile
                                            │
                            snakemake (CLI subprocess)
                                            │
            ┌───────────────────────────────┼───────────────────────────────┐
            │              │                │                │              │
       DAG resolution  staleness     cluster submission  container exec   conda
       (Snakemake)     (mtime+code)  (slurm plugin)      (apptainer/docker)
            │
            └─── per-rule run: block: shell() recipe + write_manifest()
                                            │
                                  results/<u>/<o>/...
                                  results/<u>/<o>/.lightcone-manifest.json
```

**What Snakemake owns** (we do not write code for any of this): DAG construction, topological execution, parallelism (`--cores`, `--jobs`), cluster submission (`snakemake-executor-plugin-slurm`), per-rule resources, profiles, dry-run, DAG visualization, staleness detection (`--rerun-triggers`), locking, log capture, retry, container runtime invocation.

**What we own**: a Snakefile generator, the manifest layer (write/read/verify), a status walker, and a verify routine.

## Repository Structure

```
src/lightcone/              # namespace — NO __init__.py
├── cli/                    # Click surface
│   ├── __init__.py         # exposes main()
│   ├── commands.py         # init, run, status, verify, build
│   ├── plugin.py           # get_plugin_source_dir
│   └── claude/             # force-included Claude plugin bundle (in installed wheel only)
├── engine/                 # execution substrate — Snakemake-based
│   ├── __init__.py
│   ├── manifest.py         # write_manifest, sha256_dir, code_version — the integrity layer
│   ├── snakefile.py        # generates .lightcone/Snakefile from astra.yaml
│   ├── container.py        # Content-addressed container builds (Docker, podman-hpc, apptainer)
│   ├── status.py           # Manifest-driven status walker (no Snakemake import)
│   ├── verify.py           # Recompute hashes; validate provenance chain
│   ├── tree.py             # Sub-analysis tree traversal (kept from before)
│   ├── validation.py       # Post-materialization output shape checks
│   └── site_registry.py    # Known HPC site defaults (Perlmutter, etc.)
└── eval/                   # Quantitative eval harness (top-level; peer of cli/engine)
    ├── cli.py              # `lc eval` subcommand group
    ├── harness.py, sandbox.py, graders.py, build.py, report.py, models.py

claude/lightcone/           # Claude plugin source — force-included into the wheel
├── skills/                 # lc-new, lc-from-code, lc-from-paper,
│                            # lc-feedback, ralph;
│                            # paper-reproduction bundle: lc-from-paper (entry),
│                            # ralph (loop substrate), narrative,
│                            # paper-extraction, figure-comparison,
│                            # check-sentence-by-sentence
│                            # (see skills/README.md for the full bundle map)
├── agents/                 # lc-extractor
├── templates/              # Project CLAUDE.md template
└── scripts/                # Session hooks (bash): venv activation, validate-on-save, session-start primer

tests/                      # pytest — mirrors src/ structure
pyproject.toml              # hatchling + hatch-vcs, ASTRA + Snakemake as deps
```

## Development Commands

```bash
uv sync --group dev   # installs pytest, ruff, mypy
uv run pytest
uv run ruff check src/ tests/
uv run mypy src/
```

A `justfile` is available for common tasks — run `just` to see all recipes:

```bash
just test          # run pytest
just lint          # ruff + mypy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LightconeResearch/lightcone-cli](https://github.com/LightconeResearch/lightcone-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
