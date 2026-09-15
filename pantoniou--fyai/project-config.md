---
trigger: always_on
description: enables it (`FYAI_FYMM_PALETTE`) only when the libfymermaid header has the
---

# CLAUDE.md

Use this file when you change this repository.

## Project

`fyai` is a stateless AI coding assistant written in C. It does not use a
daemon. One invocation runs one complete tool-use loop. It commits the
canonical state and then exits. An interactive invocation owns its terminal UI
only while its process runs.

### Architecture rules

- Do not add a daemon, resident process, or hidden process state.
- Store persistent state only in content-addressed libfyaml arenas under
  `~/.fyai`.
- Keep canonical data immutable, deterministic, and address-stable between
  processes.
- Do not relocate an arena during normal operation.
- Keep tools compatible with Unix conventions. Tools read files, apply writes
  or patches, and run approved shell commands.

### Comments

Write comments in concise Technical English. State the required invariant,
ownership, or behavior. Do not narrate implementation history, restate the
code, address the reader, or use colloquial explanations. Remove a comment
when the code is self-explanatory.

## Data model

Use libfyaml generics as the native data model. Build values with functions
such as `fy_gb_mapping()` and `fy_gb_sequence()`. Do not construct JSON by
hand. Emit compact JSON only at provider boundaries with `FYOPEF_MODE_JSON`.
Parse provider responses directly into generics.

Keep provider wire data in provider-stream generics. This data includes request
IDs, tool-call IDs, finish reasons, and timestamps. Derive provider-independent
content before you calculate the canonical identity.

Use typed accessor defaults:

```c
fy_get(obj, "content", "")
fy_get(obj, "total_tokens", 0LL)
fy_get(obj, "items", fy_invalid)
```

### Generic string lifetime

Prefer `fy_castp()` to `fy_cast()`. A short string can be in the `fy_generic`
word. Thus, a pointer from `fy_cast(v, "")` can point into the local copy of
`v`. The pointer becomes invalid when the copy leaves scope. Use
`fy_castp(&v, "")` at the use site. Make sure that the generic has a sufficient
lifetime. Do not keep a cast pointer longer than its generic.

A `const char *` loop variable from `fy_foreach()`,
`fy_foreach_key_value()`, or `fy_foreach_idx_item()` is safe. The typed
accessor uses the address of the stored item. Thus, the pointer refers to
collection storage and not to a copy.

### Short forms

Use the short generic API. It says the same thing with less text:

- `fy_is_string()`, `fy_is_mapping()`, `fy_is_valid()` and the other
  `fy_is_*()` predicates, and `fy_empty()` for an empty collection or string.
  Keep `fy_generic_is_int()`, `fy_generic_is_float()` and
  `fy_generic_is_bool()`, which also test the C value range.
- `fy_foreach()`, `fy_foreach_key_value()` and `fy_foreach_idx_item()` in
  place of an index loop with `fy_get_at()` and `fy_get_key_at()`.
- `fy_any_equal(v, "a", "b")` in place of a chain of `fy_equal()` tests. It
  evaluates the value one time.
- `fy_str(v)` for a printable string and `fy_number(v, dflt)` for a number.
  `fy_str()` returns NULL for `fy_invalid`.
- `fy_stringf(gb, ...)` and `fy_join(gb, ...)` to build a string generic in a
  builder arena. To get a stable `const char *`, keep
  `fy_gb_intern_string(gb, fy_sprintfa(...))`; there is no interning format
  function.
- `fy_str_empty(s)` for a NULL or empty C string.
- `fy_mapping(gb, ...)` and `fy_sequence(gb, ...)` in place of
  `fy_gb_mapping()` and `fy_gb_sequence()`. A first argument of type
  `struct fy_generic_builder *` selects the builder path, thus the call is the
  same. Most other `fy_gb_*` functions have such a short form. The exceptions
  are `fy_gb_internalize()` and `fy_gb_intern_string()`, which have none.
  Without a builder, these forms use stack storage: do not return such a value
  from a function.

### Returned generics

A generic that a function returns must name storage that is alive after the
function. Build it in a builder that the caller supplies or that outlives the
call, or return a value that an arena holds.

- Do not return a generic that a stack builder made. `fy_mapping()` and
  `fy_sequence()` with no builder, and `fy_sprintfa()`, use the stack frame of
  the caller. They are correct as an argument to a call in the same frame.
  Do not free their result.
- Copy a C string into the builder with `fy_value(gb, s)` when the string is a
  local buffer.
- State the lifetime in the header when a function returns a borrowed value.

### Empty strings

An empty-string generic is a string. It is not null. If the style is not set,
the YAML emitter must write an empty string as `""`. The core and YAML 1.1
schemas can parse a bare empty scalar, such as `key:`, as null.

Keep empty-string configuration keys as `type: string` in
`data/config.schema.yaml`. Do not allow null to hide an emitter defect.

### Generic initialization

A zeroed `fy_generic` is not `fy_invalid`. Zero can represent an empty
sequence. `fyai_setup()` clears `struct fyai_ctx`. After this operation,
initialize each generic field explicitly.

## Source layout

- `src/main.c`: global option parsing and command dispatch.
- `src/commands.c`: verb definitions, usage output, and the main runner.
- `src/fyai.c`: engine orchestration.
- `src/fyai_sink.c`: the one rendering component and its backends.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pantoniou/fyai](https://github.com/pantoniou/fyai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
