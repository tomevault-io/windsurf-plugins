---
trigger: always_on
description: Guidance for any coding agent working in this repository. **This file is the
---

# AGENTS.md

Guidance for any coding agent working in this repository. **This file is the
shared source of truth** — everything an agent needs to work here is below,
regardless of which harness it runs in.

Harness-specific files complement it and must not restate it:

| File | Scope |
| --- | --- |
| `AGENTS.md` | Everything shared. Start here |
| `CLAUDE.md` | Claude Code specifics only — it imports this file |

Keep new project knowledge in this file. If a fact holds for every agent, it
belongs here even when you discovered it in one harness.

## What this repository is

A curated "awesome list" of EV charging protocol tools (OCPP, OCPI, ISO 15118, OICP, eMI³, OIOI, Eichrecht). Two distinct deliverables live here:

1. **`README.md`** — the published awesome list. Its prose (intro, Contents, `## Specifications`, `## Contributing`, `## Other Resources`) is hand-authored. The **project listing** inside `## Tools and Resources` — everything between the `<!-- BEGIN GENERATED PROJECTS -->` / `<!-- END GENERATED PROJECTS -->` markers — is **generated** by `python pipeline.py render --readme README.md` and injected in place. Do **not** hand-edit between those markers; edits are overwritten on the next render. To change what appears there, adjust the pipeline inputs (see Conventions) and re-render.
2. **`pipeline.py`** — a discovery pipeline that scrapes GitHub for candidate repositories, scores them with quality signals, AI-classifies them, and renders the curated project listing that populates the README's `## Tools and Resources` block.
3. **`csms.md` + `csms.py`** — a separate product-level catalogue of Charging Station Management Systems (see "csms.py" below). Independent of `pipeline.py`, which it only imports.

The repo also vendors protocol specifications as static assets under `ocpp/`, `ocpi/`, `oicp/`, `emi3/`, `eichrecht/` (PDFs, WSDLs, OCPP JSON schemas) — these are reference material linked from `README.md`.

## Commands

The project is managed with **mise** (`mise.toml`): it pins Python 3.11.16 and
auto-creates/activates a `.venv` — no manual `source .venv/bin/activate`. Run
`mise trust` once after cloning.

```bash
mise install          # install the pinned Python
mise run install      # (alias: mise run i) runtime deps into .venv
mise run install-dev  # runtime + pytest; `install` alone keeps the CI data job lean
#   A fresh .venv has no pytest — run this before `mise run test`, or the suite
#   fails with "No module named pytest" and it looks like a broken environment.
mise run test         # (alias: mise run t) pytest

mise run ingest       # Stage 1 -> repos.csv   (wires --token via `gh auth token`)
mise run enrich       # Stage 2 -> repos.enriched.csv

# Or call the script directly (mise auto-activates .venv for commands run in-dir):
python pipeline.py ingest --token <GITHUB_PAT> --out repos.csv
python pipeline.py enrich --in repos.csv --out repos.enriched.csv --token <GITHUB_PAT>
#   enrich flags: --limit N / --skip-forks / --skip-dormant / --refresh

python pipeline.py render --readme README.md   # Stage 3 -> injects the curated
#   Selection between the markers in README.md and writes the secondary
#   legacy-projects.md (dormant + to-refine). Omit --readme to only write the latter.

git show HEAD:classifications.csv | python pipeline.py check-classifications --base -
#   CI guard, see "Guarding the classification cache" below.
```

`--token` is optional; without it GitHub's unauthenticated rate limits apply.
`gh auth token` supplies one when the gh CLI is authenticated.

`mise run test` (alias `t`) runs the pytest suite in `tests/`, which covers
`csms.py` only: the controlled feature vocabulary, certificate derivation,
product/company identity, merge precedence, table rendering, the curated-CSV
validations and render determinism. It needs no network — the one GitHub call
`merge` can make is monkeypatched — and it also validates the committed CSVs, so
a bad curated row fails there as well as at render time. `tests/test_pipeline.py`
covers `pipeline.py`'s classification layer — the CI guard, the "is there
anything to classify" rule, the backends' failure contract and what `enrich`
writes to the durable cache — with the backend and the README fetch stubbed, so
it needs no network and spawns no LLM CLI. The rest of `pipeline.py` is
untested; its pure functions can be exercised by importing `pipeline` and
calling `build_repo_record` / `days_since_push` / `parse_categories` directly.

Markdown is linted by `npx markdownlint-cli2` (config in
`.markdownlint-cli2.jsonc`); `README.md` additionally by `npx awesome-lint`.

## pipeline.py architecture

A two-stage pipeline connected by a **CSV boundary** — ingestion (deterministic, GitHub-only) is deliberately separated from enrichment (the slow/flaky LLM step), so the CSV can be reviewed/curated in a spreadsheet and enrichment can be re-run without re-fetching.

**Stage 1 — `ingest` → `repos.csv`** (`ingest`, `collect_candidates`, `build_repo_record`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juherr/awesome-ev-charging](https://github.com/juherr/awesome-ev-charging) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
