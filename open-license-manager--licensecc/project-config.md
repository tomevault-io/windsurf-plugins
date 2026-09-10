---
trigger: always_on
description: Use when writing or editing C++ source files (cpp/hpp/h/cc/cxx) in licensecc. Covers RAII, value semantics, const-correctness, C++11 constraints, smart-pointer construction, exception safety across the C API boundary, and common pitfalls like dangling c_str() and temporaries bound to references.
---


# Coding Guidelines

These guidelines are for the `licensecc` library specifically — the sections below
call out where the codebase is already consistent (follow it) and where it is
inconsistent (pick the dominant convention for new code; don't add a third style).

## Repo Conventions

### Naming

- Functions and methods: `snake_case` (`verify_signature`, `get_active_strategies`,
  `generate_pc_id`) — this is the dominant style across the library. A few older
  files use `camelCase` (`readLicenses`, `mergeLicenses`, `toLicenseInfo`,
  `getAdapterInfos`); treat these as legacy, don't extend the camelCase style to
  new code, and prefer `snake_case` when touching them anyway.

### Comments

- The documentation will be rendered with Doxygen. Use its specific syntax to comment
- Do not put comments inside the methods. Only reserve for the very few cases where the code is extremely hard to comprehend (bit shifts, machine operations)
- Comment only classes declaration in .hpp files.
- Do not automatically add comments on methods if not specifically asked.

### Header guards

Use `#ifndef FOO_H_` / `#define FOO_H_` / `#endif`, matching the convention used in
the large majority of headers in this repo. A couple of files use `#pragma once`;
don't add more — pick the `#ifndef` guard style for new headers.

### `using namespace`

`using namespace std;` (and similar) is used throughout `.cpp` files in this
codebase and that's fine — but it must **never** appear in a header (`.h`/`.hpp`),
since headers are included into consumers' translation units and would leak the
namespace into their code.

### Exceptions across the C API boundary

The library throws `std::logic_error` / `std::invalid_argument` / `std::runtime_error`
internally (hardware identification, date parsing, file I/O). That's fine for
internal C++ code, but every function exported through the C API
(`include/licensecc/licensecc.h`, i.e. `identify_pc`, `acquire_license`,
`confirm_license`, `release_license`) is a hard boundary: **no exception may cross
it**, since callers may be linking from C, a different runtime, or across a DLL
boundary, where an escaping exception is undefined behavior, not a catchable error.
- `LicenseFacade::identify_pc` wraps its work in `try { ... } catch (const std::exception&)`
  and logs/degrades gracefully — this is the pattern to copy.
- `LicenseFacade::acquire_license` currently does **not** wrap its call chain
  (`LicenseParser::readLicenses`, `LicenseVerifier`) in a try/catch, even though
  those paths can throw. Don't replicate this gap in new code — any new logic
  reachable from a C API entry point needs its own top-level `try/catch`, and if
  you're touching `acquire_license`, wrapping it is an in-scope fix, not scope creep.
- Do not suggest throwing exceptions in the generated code. Suggest returning error codes instead, and let the caller decide how to handle them. If you find opportunity refactor code that throws exceptions to return error codes instead (no separate PR).

### Smart pointer construction

- This project targets **C++11** (`CMAKE_CXX_STANDARD 11` in the top-level
  `CMakeLists.txt`) — `std::make_unique` is **C++14** and is not available here.
  Construct with `std::unique_ptr<T>(new T(...))` directly; don't suggest or add
  `make_unique` calls.
- Don't `dynamic_cast` a freshly-constructed derived pointer up to its own base
  (`unique_ptr<Base>(dynamic_cast<Base*>(new Derived()))`, seen in
  `identification_strategy.cpp` and `LocatorFactory.cpp`) — the derived-to-base
  conversion is implicit and always safe at that point (a `unique_ptr<Derived>`
  converts to `unique_ptr<Base>` on its own in C++11), so `dynamic_cast` there only
  adds a needless runtime check. Just write `unique_ptr<Base>(new Derived())`.

### Buffers

Prefer `std::vector<unsigned char>` or `std::string` over raw `new[]`/`delete[]`
for byte buffers (signature buffers, hashes, raw hardware data). Where a raw buffer
is unavoidable (e.g. interop with an OS or OpenSSL API that wants a raw pointer),
scope its lifetime with a smart pointer with an array deleter or a `std::vector`'s
`.data()`, not a bare `new[]` paired with a manual `delete[]`.

### String copying into fixed-size buffers

`strncpy` is banned: it doesn't always NUL-terminate (when the source is ≥ n bytes)
and silently truncates. Use the repo's `mstrlcpy` helper (declared in
`src/library/base/string_utils.h`, a `strlcpy`-style implementation) for any copy
into a fixed-size C buffer, and pass the full destination size — it always writes
the terminating `\0`:

```cpp
// WRONG: may leave the buffer unterminated when src >= sizeof(dst)
strncpy(dst, src, sizeof(dst) - 1);

// CORRECT: always NUL-terminates, copies at most sizeof(dst) - 1 bytes
mstrlcpy(dst, src, sizeof(dst));
```

## General C++ Guidance

### 1. Resource Management

#### RAII (Resource Acquisition Is Initialization)

Unlike Java's garbage collection, C++ uses RAII for automatic resource management:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-license-manager/licensecc](https://github.com/open-license-manager/licensecc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
