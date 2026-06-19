---
trigger: always_on
description: Systematic literature-mining pipeline for discovering novel covalent reactive handles capable of forming stable bonds with specified biological side-chain functionalities (including cysteine, lysine, serine, histidine, tyrosine, threonine, aspartate, glutamate, and methionine) under intended probe-use conditions. Use when a user asks to survey recent chemistry literature for novel covalent reaction partners targeting a specific residue or functional group, search for side-chain-reactive covalent
---


# Covalent Probe Discovery

A five-step pipeline that turns a target residue or side-chain functionality
into a short, paper-style Markdown report of novel covalent reactive handles,
each accompanied by a validated SMILES, an RDKit-rendered structure, and
citations to the primary literature.

The pipeline is designed to be reproducible and auditable: every artifact
produced by a run is retained, every candidate in the final report is backed
by a locator in the source PDF, and the known-warhead exclusion list is
versioned alongside the skill itself.

## Workflow Overview

```
Step 1  Generate keywords (keyword-generation prompt)   keywords.txt
Step 2  PubMed search                                   pubmed_results.json
Step 3  Relevance scoring (relevance-scoring prompt)    shortlist.json
Step 4  Attempt OA PDF acquisition                      pdfs/, download_log.json
Step 5  Extract, validate, render, compose report       candidates.enriched.json,
                                                        images/*.png, report.md
```

Steps 1, 3, 5a, and 5c are LLM-driven and follow the prompts in
[references/](references/). Steps 2, 4, and 5b are deterministic scripts under
[scripts/](scripts/).

## Artifacts produced per run

The run directory is chosen by the caller (not the skill). Inside it the
pipeline produces:

| File | Produced by | Audience |
|------|-------------|----------|
| `keywords.txt`                | Step 1 | reproducibility |
| `pubmed_results.json`         | Step 2 | audit |
| `shortlist.json`              | Step 3 | audit |
| `pdfs/<doi>/paper.pdf`        | Step 4 | evidence source when available |
| `pdfs/download_log.json`      | Step 4 | acquisition audit |
| `candidates.json`             | Step 5a | structure-extraction prompt output |
| `candidates.enriched.json`    | Step 5b | validated candidate record |
| `images/candidate_NNN.png`    | Step 5b | figures for the report |
| `candidate_validation.json`    | Step 5b audit | JSON Schema / evidence audit |
| `report.md`                   | Step 5c | **the deliverable** |

`report.md` reads like a short paper. Confidence scores, evidence quotes,
per-claim locators, structure-source locators, and manual-review status live
in `candidates.enriched.json`; they do not appear as database fields in the
human report.

## Setup

Install this repository as a skill folder first (for example under
`~/.openclaw-autoclaw/skills/pClaw/` for OpenClaw / AutoClaw). The installed
folder must contain this `SKILL.md`, plus the bundled `scripts/`,
`references/`, and `data/` directories. The folder name may be `pClaw`; the
registered skill name is the `name` field at the top of this file:
`covalent-probe-discovery`.

### Python dependencies

The pipeline has three dependency tiers:

- Step 2 (`scripts/pubmed_search.py`) uses only the Python standard library.
- Step 4 (`scripts/download_pdf.py`) requires `httpx` and network access to
  Europe PMC, PMC's OA service, and the Unpaywall API.
- Step 5b (`scripts/smiles_to_image.py`) requires `rdkit`.

Install with pip:

```bash
python3 -m pip install -r requirements.txt
```

If `pip` cannot install `rdkit` on your platform (common on Apple Silicon
outside conda), use a conda-compatible environment:

```bash
conda create -n covalent-probe python=3.12 -c conda-forge rdkit httpx
conda activate covalent-probe
```

### Required configuration

The Unpaywall API requires a contact email. Set it before running Step 4:

```bash
export CHEM_PDF_UNPAYWALL_EMAIL="you@example.com"
```

Keep real local values in your shell profile or a private `.env` file. Do not
commit real emails, API keys, or tokens. See [env.example.sh](env.example.sh)
for the full list of optional environment variables. The skill does not
auto-load `.env` files.

### Non-package prerequisites

- Network access to Europe PMC, PMC's OA service, and the Unpaywall API.
- The bundled [data/known_warheads.json](data/known_warheads.json) exclusion
  list (shipped with this skill; override with `COVALENT_PROBE_KNOWN_WARHEADS`
  or `--known-warheads` on `run_pipeline.py` if you maintain a local copy).

No extra command-line tools are required — the pipeline does not call
`pdftotext`, `poppler`, `ghostscript`, `openbabel`, `java`, or `node`.

## Running the pipeline

All examples below assume the current working directory is the run directory
and `$SKILL_DIR` points to the skill root:

```bash
export SKILL_DIR="/absolute/path/to/pClaw"
```

An orchestrator for the deterministic steps (2, 4, 5b) is provided:

```bash
python3 "$SKILL_DIR/scripts/run_pipeline.py" --keywords keywords.txt
```

See `python3 "$SKILL_DIR/scripts/run_pipeline.py" --help` for options. The
sections below document each step individually.

## Step 0 — Confirm survey scope with the caller


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mr-johnee/pClaw](https://github.com/mr-johnee/pClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
