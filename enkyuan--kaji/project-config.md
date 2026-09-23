---
trigger: always_on
description: Handles low-risk parallel work such as fixtures, repetitive migrations, documentation synchronization, formatting, scaffolding, and CI wiring.
---

# AGENTS.md

## Purpose

This repository is optimized for small, high-quality SDKs that humans can understand, modify, and ship without depending on any specific coding agent.

Agents are expected to operate like senior engineers: reduce scope, preserve invariants, make failure modes explicit, and leave the codebase simpler than they found it.

The repository is the source of truth. Important decisions must exist in code, tests, ADRs, or handoff files — never only in chat history.

## Repository shape

```text
.
├── AGENTS.md
├── apps/
│   └── docs/                 # public documentation site
├── packages/
│   ├── ts/                   # reference implementation
│   ├── py/                   # language port/scaffold
│   └── go/                   # language port/scaffold
├── docs/
│   ├── product.md            # what the product is / is not
│   ├── architecture.md       # boundaries and dependency direction
│   ├── invariants.md         # guarantees that must remain true
│   ├── api.md                # intended public API
│   ├── decisions/            # ADRs: durable design decisions
│   ├── playbooks/            # release/debug/maintenance procedures
│   ├── research/             # external findings and comparisons
│   └── work/
│       ├── active.md         # current repo-wide objective
│       └── handoffs/         # task state transferred between agents
├── examples/                 # executable proof of product value
├── tooling/
│   ├── checks/               # structural/API/repository checks
│   └── scripts/              # narrow automation only
└── .github/workflows/
```

Do not add a new top-level directory unless the responsibility cannot fit an existing one cleanly.

## Engineering standard

Optimize in this order:

1. correctness
2. simplicity
3. readability
4. explicit failure behavior
5. maintainability
6. performance
7. extensibility

Do not optimize for hypothetical future requirements.

Prefer deletion over compatibility machinery when there is no demonstrated user requirement. Use platform primitives before inventing project abstractions.

A strong implementation should be understandable from the public types, filenames, and tests before reading internal details.

## Feynman test

Every public abstraction must be explainable in 2–3 plain sentences.

If an abstraction cannot be explained without introducing several other abstractions, it is probably too large or at the wrong layer.

Before adding a type, class, interface, subsystem, or directory, answer:

- What concrete problem does it solve?
- Why can an existing primitive not solve it?
- What invariant does it own?
- What code becomes simpler because it exists?

If the answers are weak, do not add it.

## Code shape

Prefer small pure functions, narrow interfaces, explicit inputs and outputs, immutable values where practical, dependency injection at stable boundaries, one obvious execution path, deliberate error handling, and boring control flow.

Avoid hidden global state, deep inheritance, generic framework layers, speculative abstractions, duplicate execution paths, boolean parameter piles, vague “manager/service/engine” types, and wrappers that only rename another API.

A function should usually fit on one screen. A file should usually stay within 150–300 LOC and must justify exceeding 400 LOC.

## Source organization

The filesystem should communicate product architecture. A contributor should locate behavior from filenames before reading implementation.

### Root source files

Keep primary public concepts at `packages/ts/src/` while their implementation remains small:

- `capability.ts`
- `schema.ts`
- `kaji.ts`
- `errors.ts`
- `index.ts`

`index.ts` is the only package barrel and contains no implementation.

### Responsibility directories

Create a directory when one concrete responsibility owns multiple cooperating files and grouping makes ownership clearer. Keep a public concept at `src/` root while it remains small; a directory should usually own roughly three or more cooperating files before it exists.

Current responsibilities:

- `execution/` — one capability invocation through Kaji
- `store/` — execution claim, settlement, replay, and storage

Do not organize source around generic architectural labels such as `core/`, `internal/`, `services/`, `types/`, `utils/`, `helpers/`, `common/`, or `shared/`.

### Filenames

Inside a responsibility directory, do not repeat the directory name in the filename. Prefer `execution/context.ts`, `execution/result.ts`, and `store/memory.ts`; avoid `execution/execution-context.ts` and `store/memory-store.ts`. Filenames should identify the concrete concept they own.

### Types

Types live beside the concept that owns them. Do not create a general `types/` directory. Execution result types belong in `execution/result.ts`; store contract types belong in `store/store.ts`.

### Barrels

Do not create nested barrel files by default. Avoid `execution/index.ts` and `store/index.ts`; internal imports name the owning file explicitly. `src/index.ts` alone defines the package's public API.

### Dependency direction within a package

Source placement implies ownership and dependency direction.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enkyuan/kaji](https://github.com/enkyuan/kaji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
