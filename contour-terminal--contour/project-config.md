---
trigger: always_on
description: manages to be both a defaulted `bool` parameter and a `bool` return.
---

# C++ Coding Guidelines

- Prefer C++23: `constexpr`, `std::ranges`, `std::format`, `std::expected`, structured bindings
- C-style loops are forbidden; use range-based for loops exclusively
- Use `std::views::iota` and other views for generating and transforming ranges
- Use `std::span` for passing arrays and contiguous sequences
- Use `auto` type deduction to improve readability
- Use `const` correctness throughout (refs, pointers, member functions)
- Mark return values `[[nodiscard]]` where ignoring the result would be a bug
- Document new public functions, classes, structs, and their members using Doxygen style:
  ```cpp
  /// Short description of the function (be concise).
  /// @param name Description.
  /// @return Description.
  ```
- **Header files use the `.hpp` extension.** `.h` is reserved for what is genuinely a C header —
  system and third-party headers, and files shared with GLSL (`src/vtrasterizer/shared_defines.h`
  is the sole first-party example). Every header carries `#pragma once`, never an include guard.
- Naming conventions and static-analysis rules live in **`./.clang-tidy`**, the single config for
  the whole tree: it sits at the repository root so every first-party directory resolves to it by
  clang-tidy's normal parent-directory search. There are deliberately no module-local overrides —
  one style, one config. It is the authoritative source and wins over any prose here.
- **Types are `CamelCase`, everything else `camelBack`**, with a leading underscore on non-public
  data members. The exceptions are names the standard library binds to *by spelling* — member
  typedefs (`value_type`, `iterator`, `difference_type`, …), nested `iterator` classes, container
  methods (`begin`, `push_back`, …) and `std::formatter`/`std::hash` specializations. Each such
  name carries a `// NOLINT(readability-identifier-naming)` at its declaration.
- C++ code formatting rules are defined in `.clang-format`; run `clang-format` after changes.
- Use smart pointers for ownership; do not use raw owning pointers
- Do not introduce new third-party dependencies without strong justification
- Do not suppress clang-tidy warnings with `NOLINT` comments; fix the underlying issue

## Design Patterns & Principles

Always aim for a clean software architecture. The following principles are load-bearing and
should be adhered to unless there is a very strong, explicitly justified reason not to.

### Error handling: `std::expected<T, E>`
Prefer `std::expected<T, E>` for fallible API surface. Give each subsystem its own error enum,
introduced *as the need arises* — do not invent a taxonomy up
front. Chain monadically with `and_then`, `or_else`, `transform`, `transform_error` rather
than nested `if`s. Reserve exceptions for programmer errors (precondition violation, contract
misuse), not for expected, recoverable failures.

### `enum class` over `bool`
**A `bool` in an API is an anonymous enum whose two values are named after their representation
instead of their meaning.** A `bool` parameter, return type, or data member is a finding unless one
of the exceptions below applies; the replacement is a purpose-named `enum class`. The tree already
has dozens of these and they are the model to copy: `WrapPending`, `HighlightSearchMatches`,
`JumpOver`, `StatusLineStyling`, `PixelReporting`, `ClusterWidthPolicy`, `GlyphWidthPolicy`.

The parameter case carries all three costs at once:

- **The call site loses the meaning.** `Line::toUtf8Trimmed(true, false)` tells a reader nothing,
  and no amount of careful naming *inside* the function repairs the code that calls it.
- **The compiler stops helping.** `bool` accepts pointers, integers and characters through implicit
  conversion, so an overload taking `bool` can quietly swallow an argument meant for another one,
  and two adjacent `bool` parameters can be exchanged without a diagnostic. An `enum class`
  converts from nothing.
- **A third case rewrites every signature.** When yes/no becomes yes/no/inherit, a `bool` forces a
  signature change and an edit at every call site; an `enum class` gains an enumerator and `switch`
  exhaustiveness names the places that must now handle it — the same argument data-driven design
  makes.

`TerminalSession`'s guarded-role API is the case to learn from: `applyPendingFontChange`,
`applyPendingPaste`, `executePendingBufferCapture` and `executeShowHostWritableStatusLine` all take
`(bool allow, bool remember)`, as does `executeRole(GuardedRole, bool, bool)`. The two arguments
are adjacent, identically typed and silently exchangeable, and together they gate a *permission*
decision — swap them at one call site and you grant what should have been remembered. Also
`Terminal::programUDK(bool clearAll, bool locked, …)`, `tabVisualStateFor(bool active,
bool hovered, bool windowActive)`, and `Terminal::refreshRenderBuffer(bool locked = false)`, which
manages to be both a defaulted `bool` parameter and a `bool` return.

**The shape of the fix.**

- **Name the enum after the decision, not after the type.** Domain words beat `Yes`/`No`
  (`ClusterWidthPolicy::FirstCodepoint`); reserve `Yes`/`No` for a type whose own name already
  reads as the question, as `JumpOver::Yes` and `HighlightSearchMatches::No` do.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [contour-terminal/contour](https://github.com/contour-terminal/contour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
