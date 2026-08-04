---
trigger: always_on
description: Guidance for working in the Algraf repository.
---

# CLAUDE.md

Guidance for working in the Algraf repository.

## What Algraf is

Algraf is a block-scoped, algebraic grammar-of-graphics DSL (file extension
`.ag`). It parses a declarative chart description, validates it against CSV
data, trains scales, and emits deterministic SVG. The whole system — parser,
language server, runtime, and renderer — ships as one Rust binary named
`algraf`.

## General DSL and API approach

Keep Algraf's language surface declarative and visualization-focused. The DSL
may name data sources, columns, stats, scales, guides, marks, and rendering
options; it must not grow awareness of producer languages such as PDL, concrete
data engines such as Polars, or storage details such as Arrow arrays and
Parquet row groups.

Use strict crate boundaries when adding performance work:

- `algraf-data` owns concrete loaders and conversion details for native formats.
- `algraf-driver` owns caller-data source resolution, format selection,
  sniffing, reader construction, and unsupported-format diagnostics.
- `algraf-render` owns visual semantics and consumes `Table`, typed column
  views, scan helpers, handles, or aggregate APIs. It must not import Arrow,
  Parquet, Polars, SQLite, or other native data-engine symbols.
- Parser, syntax, semantics, editor services, ordinary LSP surfaces, and public
  browser/WASM APIs must stay independent of concrete native data engines.

Native performance features are CLI/native concerns unless a plan explicitly
says otherwise. Do not enable Arrow stream, Parquet, Polars, SQLite, native
filesystem, or similar data-engine support in `algraf-wasm`; browser callers
should receive stable unsupported-format diagnostics for native-only formats.

## Spec and versioned plans

Three artifacts govern behavior, and they must stay in sync:

1. **`docs/ALGRAF_SPEC.md` — the normative reference.** It describes what the
   implementation *does*, not just what it might do. **Read the relevant section
   before implementing or changing behavior.** The spec uses RFC-2119-style
   keywords (`MUST`, `SHOULD`, `MAY`, `MUST NOT`) and assigns stable diagnostic
   codes (e.g. `E0012`); honor both. When code intentionally deviates from a
   `SHOULD`, document why in a comment.

2. **`docs/V0_<minor>_PLAN.md` — per-release planning files.** Each release gets
   one (e.g. `V0_2_PLAN.md`, `V0_3_PLAN.md`). A plan states the release thesis,
   lists Must/Should items with a `Status:` line each, and records what stays
   deferred. Plans are *guidance*, not normative: a feature is only real once the
   spec says `MUST`/`SHOULD` and the code implements it. The earliest
   unreleased plan is the active implementation target; later unreleased plans
   are sequencing guidance, and completed release plans are historical records —
   don't reopen a completed release's scope.

3. **The code** under `crates/`, plus its tests and `examples/`.

### How they tie together

- **Promoting a deferred feature** (an old `MAY`, "later versions", or "deferred"
  item) into a release: add it to the active plan file, then move it into the
  relevant normative spec section (`MUST`/`SHOULD`), reserve any new diagnostic
  codes in the spec *before* implementing, then implement + test + add an example.
  See the Promotion Workflow at the bottom of each plan file.
- **The spec must match the implementation, not run ahead of it.** If you ship a
  geometry, property, theme, scale/guide key, CLI flag, or diagnostic, document
  it in the spec in the same change. If you defer something, the spec must say so
  (`MAY`, "deferred") rather than describe it as though it exists. Diagnostic
  codes emitted by code must exist in the spec; the spec MAY reserve codes the
  code does not yet emit.
- **Keep the language-reference templates in sync.**
  `crates/algraf-cli/templates/ALGRAF_LANG.md` is the full agent-facing language
  reference that `algraf init` writes into a project, and it is a tracked
  language-surface artifact like the TextMate grammar and `README.md`. When the
  split language-reference templates exist, treat
  `crates/algraf-cli/templates/ALGRAF_LANGUAGE.md` as the source for Algraf
  syntax, declarations, algebra, geometries, properties, scales, guides, themes,
  enum values, data-source forms, and rendering semantics; treat
  `crates/algraf-cli/templates/ALGRAF_TOOLING.md` as the source for CLI
  commands, caller-data workflows, package/WASM APIs, agent setup, workflow
  guidance, and operational pitfalls. The full `ALGRAF_LANG.md` must be composed
  from those parts and must not drift. Whenever you add or change a geometry,
  geometry property, scale/guide key, theme token, chart argument, enum value,
  grammar shape, CLI flag/subcommand, WASM API, package API, or generated agent
  workflow, update the relevant template source section in the same change (for
  example language sections such as "Complete Declaration Reference", "Complete
  Geometry Property Reference", and "Property Value Forms And Enums", or tooling
  sections such as "CLI Commands" and "Project Agent Setup"). Like the spec, the
  templates document only the *implemented* surface — update them alongside
  implementation, never ahead of it, so a project on the current binary cannot be
  told about syntax or tooling that errors.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [williamcotton/algraf](https://github.com/williamcotton/algraf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
