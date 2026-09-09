---
trigger: always_on
description: handles** — read a node's explicit fields, do not delimiter-split.
---

# CLAUDE.md

Agent guidance for `codellm-devkit/codeanalyzer-python`.

Respect the global `~/.claude/CLAUDE.md` instructions strictly.

`AGENTS.md` is a symlink to this file — one source of truth for every agent tool.

## Repo rules

- **Never add AI/Claude authorship anywhere** — not in commit subjects or bodies
  (no `Co-Authored-By`, no "Generated with …", no 🤖 trailer), not in PR/issue
  text, code comments, docs, or any file written to disk. This is absolute and
  overrides any tool suggestion or template.
- Use Conventional Commits (`type(scope): summary`).
- This repo's own `CLAUDE.md`/`AGENTS.md` and `.claude/SCHEMA_DECISIONS.md` are
  tracked past a global gitignore via `!`-negations in `.gitignore`; keep those
  negations if you touch the ignore file.

## Querying the emitted graph

`docs/skills/analyzing-canpy-graphs/` is a reference skill (tool-neutral, repo-shared)
for querying the analyzer's own output — vocabulary tables and recipes for entrypoint,
taint, exit-point, and slicing queries over `analysis.json` and the Neo4j projection.
Read it before writing Cypher or JSON traversals over schema v2.

## Schema v2 — the model this analyzer emits

`codeanalyzer-python` emits **canonical schema v2** (`schema_version` `2.0.0`): one
additive Code Property Graph (CPG) tree, exposed as **four gated analysis levels**
(`-a 1|2|3|4`) across two projections. Read `.claude/SCHEMA_DECISIONS.md` for the
decision log and `docs/superpowers/specs/2026-07-07-schema-v2-four-levels-design.md`
for the full model rationale. This section is the short version so a future agent
does not re-derive it.

### Additive paradigm

There is **one tree**, grown one layer deeper per level. Each level only *adds*
nodes and edges — nothing is removed or renamed between levels. The single
exception is one sanctioned refinement: a `call` node's `callee` goes `null → id`
when the call graph resolves it (L1→L2). Hence the monotonicity invariant, which
is a CI gate:

```
analysis.json(-a 1) ⊆ analysis.json(-a 2) ⊆ analysis.json(-a 3) ⊆ analysis.json(-a 4)
```

(superset modulo the `callee: null→id` refinement, and the DDG widening where L4
*adds* `prov:["points-to"]` edges over L3's `prov:["ssa"]` subset).

### Node tree + edge overlays

The payload root is the `Analysis` envelope (`codeanalyzer/schema/py_schema.py`):

```
Analysis                         # envelope: schema_version, language, max_level,
│                                #   analyzer{name,version}, k_limit (L3+ only), application
└─ application (PyApplication)   # id, kind:"application"
   ├─ symbol_table: {<file>: PyModule}         # the node tree
   │    PyModule    → id, kind:"module", source, types{…}, functions{…}
   │    PyClass     → id, kind:"class", span, callables{…}, types{…}
   │    PyCallable  → id, kind, span, body{…}, callables{…}, types{…},
   │                  cfg[], cdg[], ddg[], summary[]
   ├─ call_graph: [PyCallEdge]    # {src, dst, prov, weight} — the list name IS the type
   ├─ external_symbols: {<id>: PyExternalSymbol}  # edge-endpoint id homes (L2)
   ├─ param_in:   [ParamEdge]     # cross-function overlay, app scope (L4)
   └─ param_out:  [ParamEdge]     # cross-function overlay, app scope (L4)
```

Intra-callable graphs (`cfg`/`cdg`/`ddg`/`summary`) hang off each callable; the
truly cross-function overlays (`call_graph`/`param_in`/`param_out`) live at
application scope because their endpoints span callables.

**Keystone containment vocabulary (issue #98).** The containers use the shared
canonical names — `PyModule.types`/`.functions`, `PyClass.callables`/`.types`,
`PyCallable.callables`/`.types` — never per-language renames, so one SDK model
set parses every analyzer's output. (The historical `classes`/`methods`/
`inner_classes` names are gone as of 1.0.0.)

**No dangling edge endpoints.** Every `call_graph` endpoint joins the id space:
declared callables by their `can://` tree id, imported/builtin targets by a
`can://<app>/@external/<module>/<name>` id homed in
`application.external_symbols` (keyed by that id, `kind:"external"`).

### Identity

- **Durable `can://` ids** (`codeanalyzer/schema/ids.py`), for every node at or
  above a callable:

  ```
  can://<app>/python/<file>/<type-path>/<callable-sig>
  ```

  `<app>` is the **outermost** segment and the language sits inside it, so
  `can://<app>` is a prefix of every id this analyzer mints for that
  application — code, `@external` homes, and artifacts
  (`can://<app>/artifact/<path>`) alike. Never read the language off the first
  segment: an application named `python` yields `can://python/python/…`.
  `<app>` = `--app-name` (default: input dir name); `<file>` = the symbol-table
  key (relative POSIX path, and it *contains* `/`); `<type-path>` = nested class
  names joined by `/`; `<callable-sig>` = `name(argnames)`. Ids are **opaque
  handles** — read a node's explicit fields, do not delimiter-split.
- **LOCAL ordinal ids** (< callable), used as `body` map keys and as every
  intra-callable `cfg`/`cdg`/`ddg`/`summary` edge endpoint: `"line:col"` for real
  statements, `"@entry"`/`"@exit"` for the CFG bookends, `"@formal_in:0"` /
  `"@formal_out"` for formals, and `"<callsite>/actual_in:0"` /
  `"<callsite>/actual_out"` for actuals (parented to their call site). The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codellm-devkit/codeanalyzer-python](https://github.com/codellm-devkit/codeanalyzer-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
