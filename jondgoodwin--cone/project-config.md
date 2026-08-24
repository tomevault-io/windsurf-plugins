---
trigger: always_on
description: Cone is an in-development systems programming language. This repository
---

# Cone repository instructions

## Project overview

Cone is an in-development systems programming language. This repository
contains its C compiler (`conec`) and a small standard-library component
(`conestd`). The compiler targets LLVM and currently depends on LLVM 13.

## Repository layout

- `src/c-compiler/parser/`: lexer and parser; converts Cone source into IR.
- `src/c-compiler/ir/`: shared IR plus semantic analysis.
  - `exp/`: expression nodes and lowering.
  - `stmt/`: declaration and statement nodes.
  - `types/`: type representation and type rules.
  - `meta/`: generics and macros.
- `src/c-compiler/corelib/`: compiler-defined core language types and methods.
- `src/c-compiler/genllvm/`: LLVM type, statement, expression, and allocation
  generation.
- `src/c-compiler/shared/`: diagnostics, memory, file, option, timer, and UTF-8
  utilities.
- `src/conestd/`: the C implementation of the standard-library component.
- `conesite/`: source and static content for
  [cone.jondgoodwin.com](https://cone.jondgoodwin.com), including the
  playground, examples, and language documentation. The reference
  documentation manifest is `conesite/public/coneref/index.html`.
- `design/`: design notes, grouped as `northstar/` (what Cone is aiming at and
  how far the compiler is — the notes that would survive a rewrite), `phases/`
  (one per compiler phase, plus the naming rules), `nodes/` (what is true of
  every IR node, plus per-node notes), `compiler/` (how `conec` itself is built
  and stays fast), and `diagnostics/` (measuring, error codes, test suite). Use
  `design/_index.md` to find the relevant topic, then page in only the notes
  needed for the task.
- `workitems/`: active and backlog compiler/language work. Its
  `workitems/_index.md` plan summarizes the work and acts as the manifest for
  the individual work-item notes; `workitems/__top-priority.md` identifies the
  current priority sequence.
- `test/run.py`: the test suite runner. `design/diagnostics/test-suite.md` is its authoring
  guide.
- `test/cases/<group>/`: one directory per coverage group, each with a
  `cases.toml` listing its scenarios.
- `test/codes.toml`: the pinned `ErrorCode` name-to-number table the runner
  checks `error.h` against before any case runs.

## Documentation context

- Treat source code as the truth for current compiler behavior.
- Consult `design/_index.md` when a task needs design intent or subsystem
  context. Design notes complement the implementation and may describe
  incomplete or planned behavior.
- Consult `workitems/_index.md` for planned work, dependencies, and links to
  detailed active or backlog items.
- Consult `conesite/public/coneref/index.html` for the language reference
  page index and the surrounding `conesite/` files when changing published
  language documentation or playground behavior. Its chapter list is also the
  spine of the test suite's group organization: adding a chapter implies asking
  whether a coverage group is needed, and a feature with no chapter has nowhere
  to be tested.

## Compiler pipeline

`src/c-compiler/conec.c` defines the high-level pipeline:

1. Parse source into heterogeneous `INode` IR nodes — `design/phases/parse.md`
2. Resolve names — `design/phases/name-resolution.md`
3. Type-check and infer types while lowering syntactic sugar —
   `design/phases/type-check.md` for when a declaration is checked,
   `design/phases/type-check-reasoning.md` for what the checks decide
4. Run data-flow analysis from function type checking — `design/phases/flow.md`
5. Generate LLVM IR and output — `design/phases/generation.md`

Each phase note carries its key principles, what the phase deliberately does
*not* do, its hazards, and a file-and-function map into the code. Read the one
that owns the problem before changing it. `design/nodes/_index.md` covers what is
true of every IR node regardless of phase.

IR nodes may be replaced or lowered during name resolution and type checking.
When changing a language feature, trace and update every affected phase:
lexer/parser, IR node construction, name resolution, type checking/lowering,
flow analysis when ownership or borrowing is involved, LLVM generation, and
the Cone smoke-test input.

## Code conventions

- Follow the existing C style and nearby naming patterns.
- Node structs share `INode` headers and are dispatched primarily by `tag`.
- Use existing node constructors, traversal macros, interned names, namespace
  lookup, type comparison, coercion, and error-reporting helpers.
- Preserve source location data when injecting or replacing nodes so
  diagnostics remain useful.
- Keep type safety explicit; do not hide invalid IR states with unchecked casts
  or placeholder values. Pass and return the declared enum (`TypeCompare`,
  `SubtypeConstraint`, `OverloadMatch`) rather than a bare `int`.
- Add comments only where compiler lowering, ownership behavior, or LLVM
  representation is not self-evident.
- Give a diagnostic its own `ErrorCode`. Do not reuse an unrelated code for a
  new condition; lookup failures, visibility errors, and no-match errors must
  stay distinguishable.

## Build

### Windows

The verified configuration uses a 64-bit LLVM 13 installation with the X86 and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jondgoodwin/cone](https://github.com/jondgoodwin/cone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
