---
trigger: always_on
description: Knowledge forest navigable by an SLM: markdown + indexes, traversed through
---

# MonkeyLLM agent guide

Knowledge forest navigable by an SLM: markdown + indexes, traversed through
**Vine**'s MCP primitives. `docs/monkeyllm-spec-v0.74.md` is normative
(earlier versions are archived) **the spec is the truth**; any contract
change requires a new spec version before code.

## Language policy

**English is the project's native language** code, comments, docstrings,
tests, docs, CLI output, task files. When touching a file, translate any
Portuguese remnants you find in it (boy-scout rule; full sweep is tasks/T02).
**Every contract token is English** (spec v0.5): node types
(`branch`/`note`/`document`/`entity`/`concept`/`event`/`media`), rels
(`part-of`, `related-to`, `discovered-shortcut`, …), `entity_kind`/`source`
enums, and the parsed index headings (`## Sub-branches`, `## Direct bananas`,
`## Cross trails`, `## Query manual`). The Portuguese tokens were removed —
never reintroduce them.
There are no Portuguese exceptions all content (node IDs, titles, summaries,
bodies, tags, question sets, system prompts, generator literals) is English.
Forests are never edited in place change the generator and rebuild.

## Licensing

Two licenses (see `LICENSING.md`): **Apache-2.0** for the engine and
everything around it, **AGPL-3.0-only** for the host (`apps/station/`,
`apps/studio/`, `apps/clipper/`). Every new source file carries an SPDX header naming the
license of the tree it lives in. Apache-2.0 is one-way compatible with
AGPL, so the direction is load-bearing: the host may import the engine, and
`src/monkeyllm/` **must never import from `apps/`** that is now a legal
boundary, not only the Part J "privileged client" design rule.

## Layout

- `src/monkeyllm/` the `monkeyllm` package, `vine` CLI. `vine.py`
  (10 primitives), `harvest.py` (C.6c composite MCP tool: one-shot zero-LLM
  retrieval), `gardener.py` (Part G ingest: adopt/sync + pluggable
  converters), `ranger.py` (Part H maintenance: evaporation, link
  promotion/pruning, health), `catalog.py` (SQLite + FTS5 = locate's BM25
  side + scan),
  `canopy.py` (optional vector layer, Phase 1), `parser.py`/`models.py`
  (frontmatter), `forest.py`/`gitops.py` (files + commits),
  `telemetry.py`/`trails.py` (traces + pheromone).
- `forests/` ALL generated forests live here, fully gitignored except
  `forests/scripts/` (the generators: `build_fixture.py`,
  `build_bench_forest.py`, `build_dump.py`). `forests/forest-fixture/` =
  test forest (82 nodes, 12 branches, 1 SQLite dataset, own embedded git);
  `forests/bench-forest/` = Monkey Bench corpus; `forests/dump-ingest/` +
  `forests/_measure-forest/` = curation measurement. Rebuild, never edit.
- `examples/` how-to-use material. `examples/demo/` = agent↔Vine loop
  for the multi-hop questions (criterion F.5); `harvest.py` is the CLI
  wrapper over `monkeyllm.harvest`.
- `bench/` Monkey Bench: chunker, RAG baselines (topk/iter), runner.
- `scripts/` infra + measurement: `setup_models.py`, `serve_llm.py`,
  `bench_locate.py`, `measure_curation.py`, `convergence.py`,
  `junit_to_html.py`.
- `tasks/` backlog, one file per task (see `tasks/README.md`).
- `_derived/` is disposable and rebuildable (`vine reindex`); never a source
  of truth.

## Commands

```powershell
.venv\Scripts\python.exe -m pytest -q          # suite (must stay green)
python -m monkeyllm.cli init --forest D:\path --title "..."   # new empty forest
python -m monkeyllm.cli validate --forest forests\forest-fixture
python -m monkeyllm.cli reindex  --forest forests\forest-fixture
python -m monkeyllm.cli canopy build --forest forests\forest-fixture  # vector layer
python -m monkeyllm.cli adopt D:\dump --forest D:\forest     # Gardener: mirror a tree
python -m monkeyllm.cli sync --forest D:\forest              # Gardener: hash-diff refresh
python -m monkeyllm.cli ranger --forest D:\forest            # Ranger: evaporate+tend+health
python forests/scripts/build_fixture.py                         # rebuild the fixture
python scripts/bench_locate.py                                  # quality+latency
```

Local models (llama.cpp on the 3090): see `docs/local-inference.md`.

## Conventions and pitfalls

- **A snapshot that is not one file is not a snapshot (spec Part I +
  J.13.1/J.13.2, v0.74)**: Part I opens with "A forest snapshot is ONE
  file" and that stopped being true when `--with-payloads` was added — a
  forest with a dataset needed a bundle AND a sidecar, separate at every
  step (two files written, two links, two upload fields with the second
  optional and defaulting OFF). It came apart in the field: a 1,877-node
  forest imported from a bare bundle, audit row `{"nodes": "1877",
  "payloads": "0"}`, status `ok`, two dataset passports naming a `payload:`
  that was not on the volume. Found two days later by a **404 in a
  console**. The mechanism was never broken — created and restored a
  sidecar correctly on the fixture the whole time — which is why the fix is
  not in it. **The load-bearing failure is the silence**: omitting payloads
  is legitimate (a metadata-only backup is a real artifact, and refusing to
  restore one would strand every pre-v0.74 bundle), but `payloads: 0`
  meaning both *this forest has none* and *you did not ask* is not, and a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JimmyWesley/MonkeyLLM](https://github.com/JimmyWesley/MonkeyLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
