---
trigger: always_on
description: This file is the repository-level operating context for coding agents. Detailed
---

# Repository Agent Guide

This file is the repository-level operating context for coding agents. Detailed
domain rules remain in the linked specifications; do not duplicate or invent
them from prompt context.

## Development Workflow

- Work on a focused feature branch and use a pull request. Do not merge until
  required CI is green.
- Keep unrelated user changes intact and surface unrelated defects separately.
- Use `Henrik Westerberg <henrik.westerberg@emergentwisdom.org>` for locally
  authored commits. GitHub-created merge commits may use the account's noreply
  address; merge locally when every commit must use the organization address.
- Website and frontend experiments must remain on an isolated branch and
  staging service until Henrik explicitly approves production deployment.
- Paper typography or stylesheet work must remain separate from semantic paper
  updates unless visual redesign is the stated scope.
- Any change that touches `paper/` must be built with `scripts/compile_paper.sh`,
  never bare `pdflatex`. The script regenerates the tables, the pattern-card
  appendix, the prose hash references and the stats macros, then runs bibtex and
  three pdflatex passes. Bare pdflatex leaves those stale: it once shipped 49
  outdated hash stubs in the prose and a wrong pattern count, and a stale
  generated `docs/information/audit.md` from the same class of mistake failed CI
  on the 3.12 job, which is the only job that runs the vocabulary-workflow check.

## Vocabulary Changes

Read these before editing patterns:

- `docs/guides/authoring.md`
- `docs/guides/lifecycle.md`
- `docs/specification/schema.md`
- `docs/specification/validation.md`

The database is authoritative. Copy exported JSON from `data/vocabulary/` into
`data/staging/`, edit staging, update `data/design_critique.json`, preview the
manual, then apply. Never edit canonical vocabulary exports directly.

To apply, prefer:

```bash
python scripts/apply_vocabulary_change.py
```

It runs the four steps in the only order that works and stops at the first
failure. It ignores any database selected by `sema use` and pins every
subprocess to this checkout's authoritative `data/taxonomy.db`.

**Run it as its own command, after the sidecar edit has landed.** Chaining a
`design_critique.json` edit and the apply in one shell invocation lets the
reversible step fail while the irreversible one proceeds: a script that died
part-way through writing the sidecar still allowed the apply to change four cards
and clear staging, leaving the corpus edited with the argument for it unwritten.
Note also that `design.critique` is a list on some patterns and a string on others,
which is what killed that script — handle both, or read the type first. The order is load-bearing: `sema apply` writes the database,
`export_sema.py` writes `data/vocabulary/` from the database, and
`rebuild_vocabulary.py` reads `data/vocabulary/` to recompute hashes so that
dependents of an edited pattern pick up its new hash. Rebuilding before
exporting rehashes the previous state instead, which once let a dependency cycle
survive a fix that had already been applied.

Validation refuses a cycle before anything is written, including a cycle between
a staged pattern and an already-committed one, and including one formed by a
`references` edge in each direction. Where the reverse edge already exists the
relationship is in the graph from the side that does not cycle, so name the other
pattern in prose rather than adding the edge. `rebuild_vocabulary.py --replace`
keeps the rebuilt database only when the rebuild succeeded; a failed rebuild
restores the backup, and backups are timestamped so a later run cannot destroy an
earlier one.

`scripts/audit/dangling_handles.py` reports CapitalisedNames in pattern text that
resolve to no pattern. It covers backticked names and multi-part CamelCase; single
bare capitalised words are not detectable, for reasons measured in its docstring.
Resolving a name is mechanical, but deciding whether to mint, redirect or
lowercase is not — that is Henrik's call.

General handles contain only the broad-use intersection. Put qualitatively
different strategies in descendants, quantitative identity axes in parameters,
deployment policy in callers, and contextual risks in the design sidecar.
Missing invariants or failure modes are not automatically defects. Sidecar
critique is diagnostic evidence, not a backlog of contracts to add.

## Vocabulary Review

Before proposing any pattern change, read the "Governing principles" section of
`docs/manuals/vocabulary-design.md`. It states the library's purpose before its
rules: the library is a seed, and a good seed is usable first, extensible second,
absorbable third, forkable fourth. Do not work from a summary of those rules,
including a summary written by another agent earlier in the same session.

The manual is a deliberation instrument, not a lookup table. Its per-pattern
commentary exists so a reviewer reads one pattern, thinks about it, and forms a
judgment before moving on. Bulk ranking of the corpus by a metric is the wrong
mode and will produce confident findings that fail the placement test.

The obligation on a pattern scales with fan-in, the number of patterns that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emergent-wisdom/sema](https://github.com/emergent-wisdom/sema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
