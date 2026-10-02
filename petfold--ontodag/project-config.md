---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

OntoDAG is a DAG-based associative memory and category manager. Items are placed into the DAG under supercategories; querying with a set of categories returns the intersection of their descendants. The root node `*` is the implicit ancestor of all top-level items.

Conceptually OntoDAG is a **subsumption-only ontology**: a multi-parent category lattice kept in transitively reduced form, with one query primitive — the intersection of descendant cones ("everything below *all* of these categories"). It sits deliberately between flat tags/folders (no multi-parent subsumption) and full OWL/description-logic stacks (properties, axioms, reasoning). Its distinguishing property is that the **transitive reduction of a DAG is unique**, which gives the structure a canonical form. That canonical form is what later makes it content-addressable, diffable, and mergeable — see "Planned Swarm integration" below. Keeping the invariants exact is therefore not cosmetic: they are the precondition for the persistence and multi-writer story.

## Branch history note

The `recordstore` branch was rebased onto `origin/main` (July 2026), so it now sits on top of the package restructuring (PR #5) and the earlier PRs (merge-based workflow, DOT/LaTeX export, car-market demo, Manchester-syntax OWL). The pre-rebase state — which still had the old flat layout (`dag.py`, `ontodag.py`, `loader.py` at the repo root) — is preserved on the local branch `recordstore-pre-rebase`. The legacy standalone `ontodag.py` implementation and its tests (`testontodag.py`, `testitem_ontodag.py`) were superseded by the package and no longer exist on this branch. The rebased branch was force-pushed to `origin/recordstore` on 2026-07-12, so local and remote now agree; normal pushes work from here on.

## Running tests

Tests use `pytest` from the repo root; `conftest.py` puts `src/` on the import path, so no `PYTHONPATH` fiddling is needed. Note that `testdag.py`/`testitem.py`/`testowl.py` do **not** match pytest's default `test_*.py` discovery pattern — running `pytest tests/` silently skips them, so name them explicitly:

```bash
python3 -m pytest tests/testdag.py tests/testitem.py -v    # core DAG logic
python3 -m pytest tests/test_invariants.py -v              # structural invariant tests (all 12 must pass)
python3 -m pytest tests/test_boundaries.py -v              # dependency-boundary tests (must always pass)
python3 -m pytest tests/test_cli.py -v                     # `odag` CLI (backends, set, swarm wiring via in-memory store)
python3 -m pytest tests/test_lazy.py -v              # LazyOntoDAG: eager-oracle correctness + fetch budgets
python3 -m pytest tests/test_sparse.py tests/test_multiwriter.py tests/test_cone_index.py -v  # SparseOntoDAG writer, sync merge rule, cone summaries
python3 -m pytest tests/test_dimensions.py tests/test_dimensions_dag.py -v  # parametric dimensions: grammar oracle + DAG integration
python3 -m pytest tests/test_count_deltas.py tests/test_canonical.py tests/test_is_below.py tests/test_union.py -v  # count-delta oracle (I5), canonical roots, below, get_any
python3 -m pytest tests/test_contract.py -v                 # CONTRACT.md v0.1 conformance: G1–G6 + the as-of clause, public API only
python3 -m pytest tests/test_surface.py -v                  # surface layer: rendering table, round-trip fuzz, CLI pipe rule (incl. a pty test)
python3 -m pytest tests/test_mcp.py -v                      # odag-mcp agent surface: envelope/echo/as_of/teaching errors + stdio end-to-end
python3 -m pytest tests/test_certificates.py -v             # is_below certificates: oracle sweep, tampering, cross-hash-seed verification (needs recordstore>=0.16.0)
python3 -m pytest tests/test_provenance.py -v               # provenance store: claim subjects, signed records, per-writer union (real signing gated on bee)
python3 -m pytest tests/test_prelude.py -v                  # the standard prelude: golden root v3, idempotent adoption, CLI
python3 -m pytest tests/test_units.py -v                    # registry v3 unit system: table exactness, cross-system comparisons, migration
python3 -m pytest tests/test_packs.py -v                    # graph-declared units + shipped packs: golden roots, vocabulary-travels, conflicts
python3 -m pytest tests/test_name_consumers.py -v           # every surface a NAME flows out through, against one nasty corpus
python3 -m pytest tests/test_reference.py -v                # docs/REFERENCE.md tables pinned to the code (commands, settings, kinds, MCP tools, extras, packs, API)

# Live-node CLI Swarm test — skips unless BEE_API *and* BEE_BATCH are set
# (always pass a real BEE_BATCH so nothing auto-buys; see "Bee integration status"):
BEE_API=http://<node>:1633 BEE_BATCH=<batchID> python3 -m pytest tests/test_swarm_bee.py -v
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [petfold/ontodag](https://github.com/petfold/ontodag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
