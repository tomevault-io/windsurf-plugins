---
trigger: always_on
description: Learnings and patterns for future agents working on this project.
---

# AGENTS.md

Learnings and patterns for future agents working on this project.

## Feedback Instructions

TEST COMMANDS: `uv run pytest tests/unit` (no network needed); `uv run pytest tests/integration` (hits live Neo4j Companies KG + TypeSafe API via `.env`); `uv run pytest` for both.
BUILD COMMANDS: `uv sync`
LINT COMMANDS: none configured
FORMAT COMMANDS: none configured

## Project Overview

PRIMARY LANGUAGES: Python (>=3.12)

Graph-navigation demo combining TypeSafe's `system_one` structured-decision API (`Choice` + `Noul`) with Neo4j to beam-search a path through a graph one hop at a time, driven by a free-text goal, an explicit target node, or a natural-language path-intent pattern. Targets the public Neo4j "Companies KG" demo instance by default but stays schema-agnostic (labels/properties are discovered live, never hardcoded).

## Build System

BUILD SYSTEMS: `uv` (hatchling backend), `src/` layout, package `neo4jev` under `src/neo4jev/`.

Dependencies: `neo4j-rust-ext`, `neo4j-viz[neo4j,streamlit]`, `typesafe-sdk`, `python-dotenv`, `streamlit`. Dev deps: `pytest`, `pytest-asyncio`.

## Testing Framework

TESTING FRAMEWORKS: `pytest` + `pytest-asyncio`

`tests/unit/` mocks both the Neo4j driver and the TypeSafe client — no network or credentials required. `tests/integration/` runs against the live public Companies KG (`neo4j+s://demo.neo4jlabs.com:7687`, db `companies2`, creds `companies2`/`companies2`) and a real TypeSafe call, using the same `.env` (credentials are public/non-secret, so no separate `integration.env`). The live TypeSafe test skips cleanly while `TYPESAFE_API_KEY` is empty; the Neo4j tests run regardless.

## Target Dataset (`companies2`)

The default target is the public Companies KG, **database `companies2`** (creds `companies2`/`companies2`, read-only). Live `SHOW INDEXES`:

| Index | Type | Covers |
| --- | --- | --- |
| `organization_fullName` | FULLTEXT | `Organization.fullName` |
| `person_name` | FULLTEXT | `Person.name` |
| `news_openai_small` | VECTOR | `Chunk.embedding_3_small` (1536 dims, cosine) |

15 labels. `Article` has **no** fulltext or vector index, so only `exact` lookup applies to it. `MENTIONS` runs `Article → Organization` (24.5k edges), which is why path-intent notebooks start from an `Article`.

Index names are per-database, so anything naming one is pinned to a database. The older `companies` database on the same server is laid out differently (`entity` fulltext over `Person`/`Organization` `name`, `news` vector on `Chunk.embedding`, `news_fulltext` on `Chunk.text`, and four vector indexes on `Chunk`) — that layout is what the tests and notebook 01 originally assumed, and it no longer holds. `Organization` carries both `name` and `fullName`, and the fulltext index covers only `fullName`: a fulltext hit set and a `name`-equality hit set are different node sets (fulltext "Apple" returns `Apple Music`/`Apple Ads`/`Apple Inc.`; exact "Apple" returns the three orgs actually named `Apple`).

**Candidate capping is round-robin.** `cap_outgoing_edges` applies the per-type cap first, then fills the remaining `total_cap` (default 60) budget round-robin across relationship types in name order — one edge per type per pass — so a high-edge-count type cannot starve a later-alphabet one out of the result. Apple Inc.'s 1354 outgoing edges spread over all 17 of its relationship types (1–4 edges each), and `HAS_COMPETITOR` contributes 4 of its 26 edges; before the round-robin change, that same type reported "0 of 26" because `APPLIED_FOR`…`HAS_CATEGORY` consumed the whole budget. The cap is still a *sample*: a supernode's edges that the budget left out can never be chosen, so a start node whose capped neighbourhood can express the goal is still preferable, and raising `total_cap` (a parameter, not a constant) is the lever.

## Notebooks

Notebooks under `notebooks/` are executed with their outputs saved — the saved render is the deliverable, not scratch.

The kernelspec must point at an interpreter that has **both** `ipykernel` and the project's dependencies. This is the trap: `uv run --with ipykernel` resolves "the running interpreter" to a throwaway build directory (`~/.cache/uv/builds-v0/.tmpXXXX/bin/python`) that vanishes, and `.venv` does not have `ipykernel` while `uv sync` keeps it that way. The recipe that works is to install the kernel deps into the project venv and point the kernelspec there:

```bash
uv pip install ipykernel nbclient nbformat          # into .venv; pruned again by `uv sync`
# kernels/<name>/kernel.json -> {"argv": ["<repo>/.venv/bin/python", "-m", "ipykernel_launcher", "-f", "{connection_file}"], ...}
JUPYTER_PATH=<dir> .venv/bin/python exec_notebook.py notebooks/XX.ipynb   # nbclient, kernel_name=<name>
```

`JUPYTER_PATH` entries are Jupyter *data* dirs, so `kernels/` is part of the path (`<dir>/kernels/<name>/kernel.json`). Run nbclient with `allow_errors=False` so an exception fails the run instead of being silently saved as an error output.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jexp/neo4jev](https://github.com/jexp/neo4jev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
