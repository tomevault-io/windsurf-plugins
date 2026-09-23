---
trigger: always_on
description: **welder** is a C++26 library that generates language bindings for annotated C++
---

# welder

**welder** is a C++26 library that generates language bindings for annotated C++
types by reading **C++26 reflection** (P2996) and **annotations** (P3394) at
compile time. You mark a type with attributes describing *which languages* it
should be exposed to and *which members* participate; welder reflects over it and
emits the binding registration code (e.g. pybind11 `class_<T>` calls) directly —
no external code generator, no parsing step. Targets **C++26 and newer only**.

The unit that lays those bindings down for a given framework is a **rod** (a
welding rod): a stateless policy struct, `welder::rods::<name>::rod`. You drive it
through the one public entry point, `welder::welder<Rod>`, whose static members
automate the boring, backend-agnostic boilerplate at whichever stage of the usual
hand-binding flow you want — a single type (`weld_type`), a single free function or
global/constant (`weld_function` / `weld_variable`, the semi-manual route), a
namespace into an existing module (`weld_namespace`), a namespace as a fresh
submodule (`weld_namespace_as_submodule`), or a whole module (`weld_module`) —
leaving the rest ordinary hand-written binding code. `weld_type` / `weld_function` /
`weld_variable` / `weld_namespace_as_submodule` take an optional trailing `name`
override (used verbatim, beats `weld_as`). The actual traversal lives in an injectable
**carriage** (`welder::welder`'s defaulted third template arg): the default
`welder::stitch_welding_carriage` binds only where welder's markers direct, while
`welder::tack_welding_carriage` binds an **unmarked** third-party library greedily
(ignoring the missing `weld` markers, still enforcing bindability). Each `weld_*`
entry point is a one-line forward to the carriage, so a user can also subclass
`welder::welder` to compose bespoke routines from the same gated building blocks.

**Delivery model:** **header-only** (`src/welder/…`). The vocabulary arrives via
`#include <welder/vocabulary.hpp>`; rods pull in the core themselves (`#include
<welder/rods/python/pybind11/rod.hpp>` → `<welder/welder.hpp>`). The optional C++20
`import welder;` module wrapper was **removed** until the gcc-16 `-freflection`/
modules bugs are fixed and another toolchain (Clang/MSVC) implements P2996 — see
`docs/content/header-only.md` and `.claude/context/gcc16-toolchain.md`. The
vocabulary headers are still kept std-include-free so the wrapper can return
unchanged. We also deliberately do *not* modularize internally.

**Status:** early POC, verified end-to-end (an importable Python module; a
`require`-able Lua module). Four *runtime* rods are implemented —
two **Python** (**pybind11**, **nanobind**) and two **Lua** (**sol2**,
**LuaBridge3**) — all sharing the same core and the *same* backend-neutral C++ test
cases, which each rod binds and asserts (pytest for Python, busted `.lua` specs for
Lua) as a cross-rod consistency check. The two Lua rods run the *same* busted specs
(selected by `WELDER_TEST_LUA_MODULE`). Three more are *build-time* text-emitting rods
over the same driver: **`welder::rods::luacats::rod`** reflects the welded Lua types and
emits a **LuaCATS (`---@meta`) stub file** (the Lua analogue of the Python `.pyi` stubs,
carrying the docstrings Lua has no runtime slot for); **`welder::rods::trampolines::rod`**
reflects the welded *virtual* Python types and emits a **`.hpp` of ready-to-compile,
backend-neutral pybind11/nanobind trampoline subclasses** — so a Python subclass can
override their virtuals without the trampolines being hand-written (each override splices
the base virtual's reflected types, so signatures match by construction — overloaded
virtuals dispatch per-slot, covariant overrides fold to one slot, protected NVI hooks
are covered; a C-variadic virtual with no `bind_flat` is a hard error); and
**`welder::rods::opaque_containers::rod`** reflects the welded types, finds the STL
containers they use (`std::vector<Entity>` of a welded class included, not just
scalars; and **fixed-size `std::array<T, N>` members**, reached transitively — direct,
through an opaque vector's element struct, or inherited from a non-welded trait base),
and emits a **`.hpp` of `WELDER_OPAQUE` declarations + welded aliases** that
bind them by reference — so the per-container boilerplate is not hand-written (a
namespace-scope `type_caster` specialization is a compile-time artifact no runtime rod
can emit; blanket over welded types, `by_value` opt-out, **collision-free
namespace-qualified derived names** — `vector<geo::Point>`→`VectorGeoPoint`,
`array<short,289>`→`ArrayShortIntx289` (element + `x` + extent) — overridable
per-type via an optional `transform_opaque_container(enclosing, container, member)` hook on
the name style). A **C#/.NET rod** exists as an **out-of-tree extension**
([skarndev/welder-csharp](https://github.com/skarndev/welder-csharp)): it mints its language identity from
the open `welder::user_lang` range and reuses the bare-`weld` shared test cases;
the backend-neutral machinery it motivated — `<welder/virtuals.hpp>`
(`bind_flat`/`virtual_slot`), the two-phase namespace sweep, the bare `weld`
form — stays in welder's core. Class-element
containers are made ordering-safe by the driver's **two-phase namespace sweep** (the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skarndev/welder](https://github.com/skarndev/welder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
