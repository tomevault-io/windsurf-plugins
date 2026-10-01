---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A skills-only plugin (no MCP server, no hosted backend, no vector DB) that lets Claude/ChatGPT find [Virtual Test Bed (VTB)](https://github.com/idaholab/virtual_test_bed) reactor models, explain VTB documentation, and generate or modify VTB simulation input files. Everything works from a bundled, curated static snapshot of the VTB repo (`skills/vtb-docs/references/` + `assets/`) searched by keyword at runtime — there is nothing to deploy or serve.

This directory (`vtb_assistant/`) lives inside the `virtual_test_bed` repo itself, one level below its root. `build/harvest.py` harvests directly from the enclosing checkout (its own parent directory) — there is no separate `virtual_test_bed/` clone to manage.

## Commands

```sh
uv sync --all-groups                                              # install (dev + harvest groups)

# Regenerate the knowledge pack (required before most tests do anything but skip)
uv run --group harvest python build/harvest.py
uv run --group harvest python build/build_reference_pack.py

uv run ruff check .                                                # lint (also run by CI)
uv run pytest                                                      # full test suite
uv run pytest tests/test_reference_pack.py::test_name -v           # a single test

./build/package_skill.sh                                          # zip skills/vtb-docs/ for claude.ai / ChatGPT
./build/package_plugin.sh                                         # zip the full plugin for Claude Code

uv run python build/clean.py [--dry-run]                          # remove generated build output (not .venv/.vscode/.claude)

claude --plugin-dir .                                              # load this repo locally in Claude Code
```

`harvest.py` always reads from the enclosing `virtual_test_bed` checkout (this directory's parent) — pass `--source-dir` to point it at a different checkout instead.

## Architecture

**Two-stage build pipeline, output gitignored.** `build/harvest.py` extracts raw content (doc pages + `!tag` metadata, MOOSE `tests`/`hpc_tests` spec files, open-source licensing tiers) from the enclosing `virtual_test_bed` checkout into `build/_cache/raw/*.json` — it only extracts, never curates. `build/build_reference_pack.py` turns that raw JSON into everything under `skills/vtb-docs/references/` + `skills/vtb-docs/assets/` plus `ATTRIBUTION.md`. **None of the generated output is committed** — it changes as upstream VTB evolves, so committing it would mean a large, mostly-noise diff on every refresh. Only hand-authored source is tracked: `build/*.py`/`*.sh`, `skills/vtb-docs/SKILL.md`, `skills/vtb-docs/scripts/*.py`, `commands/*.md`, `.claude-plugin/plugin.json`, `tests/*.py`. `build/RECON_NOTES.md` is scratch documentation (not shipped) recording how the pack is built and known data-quality caveats — read it before touching the extraction/curation logic.

**Runtime scripts are stdlib-only.** `skills/vtb-docs/scripts/search_docs.py`, `get_page.py`, and `get_input.py` use only `argparse`/`json`/`re`/`pathlib` — no venv, no installs, no network — because they run inside claude.ai/ChatGPT/Claude Code sandboxes with no dependency-install step available. Keep it that way; the `harvest` dependency group (`httpx`, `markdownify`, `pyyaml`) is strictly build-time-only.

**Generated pack structure** (all under `skills/vtb-docs/references/` unless noted):
- `model-index.json` — one entry per VTB model with a `!tag` block: `repo_path`/`repo_url` (pinned to the exact commit the pack was built from), `doc_url`, `codes_used_apps` vs. `tests_apps` (doc-claimed vs. test-spec-derived app usage — `apps_mismatch: true` when they don't overlap), `open_source_tier`, `is_tutorial`.
- `docs/<category>.md` — one file per top-level reactor category (not one per page — claude.ai's Skills upload caps zips at 200 files), each page delimited by a `<!-- vtb-page: <path> -->` marker. `page-index.json` gives exact byte offsets so `get_page.py` can extract a single page without scanning the whole category file.
- `model-inputs/inputs.jsonl` + `input-index.json` — an automatic per-model input-file manifest: every `.i` file under each model's own resolved `repo_path` directory (not a hand-picked exemplar per app), with content, parsed MOOSE structure (blocks/cross-references/candidate editable parameters), known test runs, provenance (`source.commit`/`source.blob_sha`/`repo_url`), and an `is_primary` flag derived from whether the model's own documentation actually names the file (run commands, markdown links, `!listing`, ...). `inputs.jsonl` holds the heavy content+parse detail (one compact JSON record per line); `input-index.json` is the lean, content-free companion `search_docs.py --kind input` searches, with byte offsets so `scripts/get_input.py` can extract one record without loading the whole file. Deliberately does not classify or bundle the files an input merely *references* (meshes, cross-section libraries, restart checkpoints) — see `build/RECON_NOTES.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [idaholab/virtual_test_bed](https://github.com/idaholab/virtual_test_bed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
