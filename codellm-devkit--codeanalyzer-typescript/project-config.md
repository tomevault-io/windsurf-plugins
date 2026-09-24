---
trigger: always_on
description: Agent guidance for `codellm-devkit/codeanalyzer-typescript` (`cants`).
---

# CLAUDE.md

Agent guidance for `codellm-devkit/codeanalyzer-typescript` (`cants`).

## What this project is

`cants` = TypeScript/JavaScript static analyzer built on TypeScript compiler
(via [ts-morph](https://ts-morph.com/)). CLDK TypeScript backend: emits
**canonical schema v2** — one additive Code Property Graph — in **two projections**,
`analysis.json` and **Neo4j** property graph. Mirrors
[Python](https://github.com/codellm-devkit/codeanalyzer-python) and
[Java](https://github.com/codellm-devkit/codeanalyzer-java) sibling analyzers, so
output-shape parity with them first-class concern.

## Schema v2 — the additive CPG (read this before touching output)

Output = **one scale-free structure**: containment tree of nodes (id / kind /
`span` / children) with **typed edge overlays** (CPG). Every classic artifact — symbol
table, call graph, CFG, PDG, SDG — is *projection* of that one structure. Analysis
**levels** = how deep it populated (each level only *adds*, never rewrites):

- **L1** (`-a 1`): tree to callable depth — `application → symbol_table{module} →
  types{}/functions{}/fields{} → callables{}` — plus `call` nodes in each callable's
  `body{}` (`callee` unresolved). `source` stored once per module; every node's
  text slices off it via `span.bytes` (UTF-8 BYTE offsets, #179 — `Buffer` slice, never `String.slice`;
  producers/consumers convert through `src/schema/offsets.ts`).
- **L2** (`-a 2`): `call_graph` edge list (callable→callable) at application scope,
  and `callee` slot on each call node refined `null → id` (only sanctioned mutation).
- **L3** (`-a 3`): rest of `body{}` (statements + `@entry`/`@exit`) and intra-callable
  edge lists `cfg`/`cdg`/`ddg` (reaching-definitions, `prov:["reaching-defs"]`) hung on each callable.
- **L4** (`-a 4`): synthetic `@formal_in:N`/`@formal_out`/`<L>/actual_in:N`/`<L>/actual_out`
  vertices, intra-caller `summary` edges, and application-scope `param_in`/`param_out`
  lists (interprocedural SDG).

**Identity two-tier**: durable `can://<app>/<lang>/<file>/<type>/<sig>` ids at callable
depth and above; ordinal `<callable-id>@<line>:<col>` (or `@<tag>`) below. Intra-callable
edge lists use **bare local ids**; cross-callable lists use **fully-qualified `can://…@local`**
ids. `L1 ⊆ L2 ⊆ L3 ⊆ L4` = CI-checkable monotonicity gate (`test/schema-v2.test.ts`).
Model + every decision live in `.claude/SCHEMA_DECISIONS.md` (§ "Schema v2 migration") and
skillset's `canonical-schema.md`.

**Provider/client boundary:** analyzer = *pure graph provider* — emits graph
substrate (CFG/PDG/SDG + `summary` edges) and stops. Slicing and taint = reachability
*queries* over it, belong to frontend SDK; never add `taint_flows` section here.

Schema v2 = **native model** (#96): stages build v2 tree directly (`src/schema/schema.ts`,
one model family — no v1 model, no emit-time reshape). Per-run passes stamp derived
layers (python parity): `assignIds` (can:// ids — per-run because ids embed `--app-name`
while cache round-trips tree), `l1Body` (`call_sites` → `body{}`), `heritage`,
`homing` + `l2Callees` (L2), `dataflow/attach` (L3/L4). `finalizeAnalysis`
(`src/schema/emit.ts`) runs them + assembles envelope + strips INTERNAL fields
(`call_sites`, `abs_path`, cache trio).

Call graph = tsc resolver + **defuse linker** (#98): deterministic per-callable
pass over resolver leftovers — alias chains, decorator edges, library-callback
edges, bounded interprocedural votes, CHA-by-name fallback. No whole-program
fixpoint, no backend flag, one code path. Module-scope calls attributed to
MODULE (python #131 parity). prov tags: `tsc` / `defuse` / `import`. Joern
superset ledger: `docs/design/specs/defuse-linker-joern-ledger.md`.

## Architecture — follow the pipeline

Whole analyzer = one orchestration function: `analyze()` in `src/core.ts`. Read
it first; everything else is stage it calls, in order:

1. **materialize** (`src/build`) — resolve/prepare target project deps.
2. **buildSymbolTable** (`src/syntactic_analysis`) — modules, classes, interfaces,
   enums, type aliases, namespaces, functions, methods, variables, decorators,
   JSDoc, with precise source spans.
3. **call graph** (`src/semantic_analysis`) — tsc resolver (`callGraph.ts`, incl.
   module-scope sweep + RTA + phantoms) then `defuseLinker.ts` tiers T1–T5;
   merged with provenance union.
4. **program graphs** (`src/dataflow`) — levels 3–4 (`-a 3`/`-a 4`): CFG → post-dominance/CDG →
   access-path def-use → PDG → SCC-condensed bottom-up summaries → SDG. This is *compute*
   (IR in `src/schema/graphs.ts`); `src/dataflow/attach.ts` writes it **onto tree**
   (`body{}` + `cfg`/`cdg`/`ddg`/`summary` per callable + `param_in`/`param_out`).
   Decisions: `.claude/SCHEMA_DECISIONS.md`; contract + staged follow-ups: issue #2.
5. **cache** (`src/utils/cache.ts`) — content-hash cache under `.codeanalyzer/`; stores
   **id-free** builder tree only (ids/body/heritage = per-run layers; levels 3–4 also
   record summaries + dependency edges in `graphs_summaries.json`).
6. **finalize + output** — `finalizeAnalysis` (`src/schema/emit.ts`, called by `analyze()`)
   runs pass spine, returns `AnalysisResult` {`application` (wire `TSAnalysis` envelope),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codellm-devkit/codeanalyzer-typescript](https://github.com/codellm-devkit/codeanalyzer-typescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
