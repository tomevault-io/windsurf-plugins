---
trigger: always_on
description: Lethe is an open-source CFD/DEM/CFD-DEM solver built on the **deal.II** finite
---

# Copilot Code Review Instructions — Lethe

Lethe is an open-source CFD/DEM/CFD-DEM solver built on the **deal.II** finite
element library (with Trilinos for linear algebra and p4est for distributed
adaptive meshes). Reviews should assume a high-performance, MPI-parallel,
template-heavy C++ codebase. Apply the rules below; cite the specific rule when
flagging an issue, and prefer a small number of high-signal comments over noise.

## Scope and tone

- Focus on correctness, parallel/numerical safety, performance, and adherence to
  Lethe's conventions. Do not re-flag formatting that `clang-format` handles.
- Do not nitpick style that `indent-all.sh` would fix automatically; assume the
  author will run it. Comment on formatting only if it suggests a logic error.
- Be concrete: point to the exact line and propose a fix.

## Code style and formatting

- Lethe follows **deal.II's clang-format style**. Indentation is enforced via
  `contrib/utilities/indent-all.sh`. Flag manually-misformatted code only when it
  hides a bug; otherwise remind the author to run the indent script.
- No `using namespace` in headers. Avoid `using namespace dealii;` at file scope
  in `.h` files.
- Prefer `const` correctness everywhere: pass large objects (deal.II `Vector`,
  `Tensor`, `Point`, matrices, `FEValues`-derived data) by `const &`, never by
  value.
- Prefer passing the whole parameters struct (e.g. a `Parameters::*` object) by
  `const &` rather than unpacking and threading many individual fields through a
  signature. Flag new/changed signatures that take a long list of scalar
  parameters already grouped in an existing struct.
- No raw `new`/`delete`. Use `std::unique_ptr`/`std::shared_ptr` or deal.II
  smart pointers (`std::unique_ptr`, `SmartPointer`) as appropriate.

## Documentation (required)

- Every function must be documented **in the header (`.h`)** with a Doxygen
  `@brief` describing what it does and a `@param` for **each** argument.
- Flag any new or modified public function missing `@brief` or with incomplete
  `@param` coverage.
- If a PR adds or changes parameters that appear in the input file (`.prm`/JSON),
  the corresponding documentation must be updated. Flag missing doc updates.

## Class and template structure

- Classes should be **implemented in `.cc`**, not inline in the header (except
  small trivial accessors).
- For templated classes, **all used instantiations must be explicitly
  instantiated at the end of the `.cc` file** (e.g. for `dim = 2` and `dim = 3`,
  and relevant scalar/vector types). Flag a new templated class whose `.cc` lacks
  explicit instantiations.

## Headers / include-what-you-use

- Include a header for **every** symbol used in a file, and **do not** include
  headers that declare no symbol used in that file.
- Flag obvious cases: a symbol used with no corresponding include (relying on
  transitive inclusion), or an include with no symbol from it in use.

## CMakeLists.txt rules

- Source and header files are listed **explicitly** in `ADD_EXECUTABLE` /
  `ADD_LIBRARY` — never via `file(GLOB)`. Flag any introduction of `GLOB`.
- When a PR adds or removes a `.cc`/`.h`, the relevant `CMakeLists.txt` must be
  updated. The file lists must be **sorted**, with **sources before headers**.
  Flag missing, unsorted, or misordered entries.
- `TARGET_LINK_LIBRARIES` must list only **direct** dependencies (those whose
  headers are actually included), and must **omit transitive** dependencies.

## Library dependency hierarchy (do not violate)

Lethe has four libraries with a strict dependency order:

- **core** depends on none of the others.
- **DEM** depends only on core.
- **solvers** depends only on core.
- **FEM-DEM** depends on core, DEM, and solvers.

Flag any `#include` that introduces a forbidden dependency — e.g. a header from
**solvers** included in **DEM**, or anything in **core** including DEM/solvers.
When in doubt, note that `contrib/utilities/checkdeps` should pass (exit 0).

## Testing (required)

- A **new feature** must be accompanied by unit tests and/or application tests,
  documentation, and updated input-file docs if parameters change.
- A **bug fix** must add a unit or application test that reproduces the bug, and
  the PR description should explain the root cause. Flag bug-fix PRs with no
  regression test.

## Release notes (required)

- Any user-facing change must add a file under `release_notes/current/` named
  `YYYY_MM_DD_LASTNAME.md`, following `release_notes/template.md`. Each entry
  carries a `MAJOR`/`MINOR` tag under the appropriate heading (Added / Changed /
  Deprecated / Removed / Fixed / Documentation). `MAJOR` covers any new feature or
  any change to a parameter name; `MINOR` covers fixes with no impact on results.
- Flag a PR that adds or changes a feature or an input-file parameter with no
  matching file in `release_notes/current/`. Do **not** edit `CHANGELOG.md`
  directly — it is assembled from these entries at release time.

## deal.II and scientific-computing correctness (high priority)

These are the issues a generic reviewer misses — weight them heavily:

- **MPI / distributed correctness**: after writing into a distributed vector or
  matrix, `compress()` must be called with the correct `VectorOperation`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaos-polymtl/lethe](https://github.com/chaos-polymtl/lethe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
