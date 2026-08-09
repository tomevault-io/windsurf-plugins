---
trigger: always_on
description: This file applies to the whole repository.  A more deeply nested `AGENTS.md`,
---

# LSQUIC Contributor Guidance

## Scope and sources of truth

This file applies to the whole repository.  A more deeply nested `AGENTS.md`,
if one is added later, overrides it for that subtree.

Before changing code, read the relevant section of `docs/internals.rst`,
and the implementation and tests around the code in
question.  Use the documentation as an architectural map, but verify details
against the current source: the internal document describes an older code
version, and the API reference itself warns that it may lag the public headers.
The current source, public headers, and CMake files are authoritative.

## Repository map and architecture

- `include/` contains the public API.  `include/lsquic.h` is its main entry
  point.
- `src/liblsquic/` contains the library implementation.  The engine owns and
  schedules connections; full connections own streams.  Server connections
  begin as mini connections and may be promoted to full connections.
- QUIC versions and connection types share interfaces but often have separate
  implementations.  Parsing and generation are selected through
  `struct parse_funcs`; gQUIC and IETF QUIC behavior must not be assumed to be
  identical.
- `bin/` contains example clients and servers and is useful for public-API
  usage patterns.
- `tests/` contains assertion-based C unit tests registered through CMake.
- `docs/` contains the Sphinx API, tutorial, development, and internals
  documentation.
- `src/lshpack/` and `src/liblsquic/ls-qpack/` are Git submodules.  Do not edit
  them unless the task explicitly includes the corresponding upstream code.

Performance is a primary design constraint.  In hot paths, preserve allocation
behavior, copy counts, batching, queue and heap complexity, packet coalescing,
and cache-friendly layouts unless a deliberate change is required.  Pay close
attention to connection refcounts and to flags that record queue or heap
membership.

When changing shared protocol behavior, determine which of these dimensions
are relevant and cover them deliberately: client/server, gQUIC/IETF QUIC,
HTTP/non-HTTP, and mini/full connections.

## Worktree hygiene

- Inspect `git status` before editing.  The tree may contain user-owned
  changes, build directories, logs, core files, `.orig` files, and `.rej`
  files.  Preserve them and do not include them in the change.
- Keep patches focused.  Do not perform broad formatting, cleanup, or adjacent
  refactoring unless it is needed for the requested change.
- Use a new task-specific out-of-source build directory; do not reconfigure,
  overwrite, or delete an existing build tree unless it was created for the
  current task.
- Do not commit generated build files or test artifacts.

## C style

Follow the surrounding file and these project conventions:

- Indent with four spaces.  Put two blank lines between function definitions
  and, generally, between all top-level constructs.
- Do not cuddle control-flow braces.  Function definitions place the return
  type on one line and the left-aligned function name on the next:

  ```c
  static int
  process_packet (struct packet_ctx *ctx)
  {
      if (ctx->ready)
      {
          /* ... */
      }
      else
      {
          /* ... */
      }
  }
  ```

- Put a space between a function name and `(` in declarations and definitions.
- Use snake_case.  Non-static functions begin with `lsquic_`.  Function names
  normally consist of a module name followed by a verb; otherwise begin with
  a verb.  Do not begin static function names with an underscore.
- Prefix structure members with an abbreviation derived from the structure
  name, matching nearby members.
- Outside the user-facing API, use `struct foo *` rather than `foo_t *` and do
  not introduce typedefs for structs, unions, or enums.  Integral typedefs are
  acceptable when consistent with existing code.
- Prefer enums to `#define` constants unless a value is larger than 2^31.  Do
  not assign explicit enum values unless the values are semantically required.
- Avoid bitfields.  Use enum bitmasks for flags and `signed char` for boolean
  structure members where appropriate.
- Use uppercase hexadecimal digits.
- Avoid ternary expressions unless the whole expression fits clearly on one
  line.
- Prefer an explicit `else` when both branches return.  This is the opposite
  of the convention commonly called `no-else-return`; the alternative without
  `else` is also described as an early-return or guard-clause style.  Write:

  ```c
  if (cond())
      return X;
  else
      return Y;
  ```

  rather than:

  ```c
  if (cond())
      return X;
  return Y;
  ```

- When several conditionals select mutually exclusive outcomes, prefer one
  visible `if`/`else if`/`else` chain over independent early returns followed
  by implicit fallthrough.  This keeps the complete branching structure easy
  to scan.  Write:

  ```c
  if (a())
  {
      some_code();
      return X;
  }
  else if (b())
      return Y;
  else
  {
      some_other_code();
      return Z;
  }
  ```

  rather than ending each independent conditional and relying on control to
  fall through to the next return.
- Early guard-clause returns are acceptable in long functions--roughly longer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [litespeedtech/lsquic](https://github.com/litespeedtech/lsquic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
