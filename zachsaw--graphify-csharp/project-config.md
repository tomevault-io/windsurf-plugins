---
trigger: always_on
description: These instructions apply to developing the Graphify C# extractor in this
---

# Contributing to Graphify C#

These instructions apply to developing the Graphify C# extractor in this
repository. They are not instructions for projects that merely use the
installed tool.

The copyable consumer skill at
`.agents/skills/graphify-csharp/SKILL.md` must stay self-contained and focused
on using the CLI against the user's codebase. Keep extractor architecture,
implementation rules, repository test commands, and release procedures here
or in the contributor documentation. Do not make installing that skill impose
these development policies on another repository.

## Working method

Read the relevant design documents under `docs/` before changing architecture.
Work in small vertical slices:

1. define or preserve a narrow contract;
2. implement the smallest independently testable unit;
3. add high-value tests for the behavior and its failure boundary;
4. run formatting/build/tests and a relevant fixture smoke test;
5. complete and verify the slice before moving on; commit when requested.

Optimize for fast, safe release. Do not add framework, dispatch, or reflection
complexity before direct semantic extraction is useful. This repository does
not require 100% code coverage; it does require focused coverage of identity,
edge direction, serialization, determinism, TFM selection, and representative
Roslyn language constructs.

### Pragmatic fast go-to-market

Treat test depth as a risk decision, not a coverage contest. For a documentation
or help-text change, use a diff check and the smallest relevant smoke check. For
isolated parsing or serialization, add focused unit tests and at least one
failure case. For semantic identity, Roslyn resolution, project/TFM loading,
edge direction, or schema changes, require focused regression tests plus an
end-to-end fixture and a determinism check. These are the areas where a small
bug can invalidate the whole graph.

Run the narrow tests while iterating so feedback stays fast. Run the full suite,
Release build, package/install smoke test, and representative real-solution
determinism check at slice or release boundaries, or earlier when the change has
high blast radius. Do not allow more than one logical slice of unverified
behavior to accumulate. Record deliberately deferred coverage or known dynamic
limitations in the relevant docs instead of silently weakening the contract.

### Performance invariants

Treat extraction cost as part of the design. Prefer Roslyn’s compilation symbol
tree for the declaration catalog, and use targeted syntax queries only for
declarations that are not exposed as type members, such as local functions,
locals, aliases, labels, and query range variables.
Reuse each project’s semantic models and source-location factory; do not reload
or reparse a project per relationship. Walk each Roslyn operation root at most
once per syntax tree and feed observations into a deduplicating edge accumulator
so overlapping operation and syntax evidence does not create a large
intermediate list. Keep fallback symbol matching O(1) on a prebuilt key index
and reject ambiguous matches without broad name scans. Measure the pinned
real-world e2e before and after semantic changes. Keep the project/target-
framework compilation as the semantic boundary, but use a bounded scheduler
with coarse, deterministic batches of source files when profiling shows
parallel extraction is beneficial and Roslyn/MSBuild thread-safety remains
clear. A source file is the smallest scheduling unit; never create work items
per class, declaration, syntax node, or edge, and do not split a syntax tree
merely to increase task count.

## Design rules

### Keep actors small

Use separate components for project loading, symbol identity, declaration
cataloging, direct reference extraction, and Graphify serialization. A component
should have one reason to change and a narrow input/output contract. Avoid
classes named `Manager`, `Helper`, or `Orchestrator` that hide several policies.
Pure records and pure functions are preferred where they make behavior obvious.

Load Roslyn/MSBuild at the boundary. Keep the domain model and output
validation usable with in-memory records and test doubles, without requiring an
IDE, Rider, or a full solution load.

### Identity must be semantic and reproducible

Never identify a C# method by filename plus method name. The key must include:

- normalized project identity;
- target framework when known;
- containing namespace and nested-type path;
- type kind/name and generic arity;
- member name and generic arity; and
- parameter types and modifiers sufficient to distinguish overloads.

Use the full canonical key as extraction evidence. If Graphify’s current node-ID
constraints require a compact ID, derive a stable ID from the key and retain the
full key in node properties. Never use a process-local hash, source line, or
unordered collection to define identity.

Roslyn can expose compiler-generated containing scopes for newer syntax. Do not
feed an empty synthetic name into the domain identity; encode a deterministic
semantic scope segment (and coalesce defining/implementing partial symbols while
retaining all source locations). If a declaration still cannot be represented,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zachsaw/graphify-csharp](https://github.com/zachsaw/graphify-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
