---
trigger: always_on
description: Rust workspace for reading, writing, and querying Altium Designer files.
---

# altium-cli

Rust workspace for reading, writing, and querying Altium Designer files.

## Workspace Structure


* **crates/altium-format-derive** Derive macros for serialization code generation
* **crates/altium-format-types** Raw types types reverse engineered from Altium
* **crates/altium-format**  Core library for Altium file parsing and manipulation
* **crates/altium-format-query**  Query interface over altium-format documents
* **crates/altium-format-render-svg**  SVG rendering for Altium documents
* **crates/altium-format-render-png**  PNG rendering for Altium documents
* **crates/altium-cli**  Command-line tool for file inspection and manipulation

## Architecture

Dependency graph:

```
altium-format-types (core types from Altium like constants, enums, and structs)
     ↓
altium-format-derive (proc macros, no runtime deps)
     ↓
altium-format (core library: parsing, querying, editing)
     ↓
altium-format-query / altium-format-render-svg / altium-format-render-png
     ↓
altium-cli (binary: CLI interface, output formatting)
```

Note: spec-language support lives in the local `altium-format-spec` crate.

**Publishing order:** types → derive → format → format-query/render-svg/render-png → cli.

**Versioning:** Synchronized versions (all crates at same version for initial releases).

**Design Philosophy**: Fail fast, fail hard. No round-trip preservation, no unknown field
capture, no opaque blobs. If our parser encounters data it doesn't understand, that is a
bug in our code that must be fixed -- never silently skipped. These files control PCB
fabrication; a silently dropped field could cost thousands of dollars.

Keep STATUS.md updated with the state of the codebase whenever you implement something.


## CARDINAL RULE: NEVER RETAIN OPAQUE FORMAT DATA

This rule is non-negotiable and overrides convenience during implementation:

- NEVER add "opaque"/"raw" retention fields for format sections or sidecars
  (examples of forbidden patterns: `opaque_sidecars`, `raw_*`, `unknown_bytes`,
  `Vec<u8>` placeholders used to carry undecoded file-format content).
- NEVER "temporarily" keep sidecar bytes for later reverse engineering.
- NEVER bypass this by claiming "source-backed save" if bytes are not fully typed and
  semantically represented in the in-memory model.
- If a stream/sidecar exists and is not understood:
  1. reverse engineer it now (C#/Delphi + fixture analysis), and
  2. implement typed parse + typed serialize, or
  3. return a hard error with full context.
- "Parse later" placeholders are forbidden in merged code.

Pre-merge self-check for agents:
- Search touched code for forbidden escape hatches:
  `opaque|raw_payload|unknown_bytes|unparsed|todo.*sidecar|Custom.*opaque|retain.*bytes`
- If found, remove by implementing typed support or restoring hard fail-fast behavior.

**NEVER skip or suppress unconsumed data**: Do NOT mark streams, records, or fields as
"consumed" without actually parsing them. This includes any form of "skip_known",
"ignore_remaining", or marking entries as consumed in `TrackedCfbDocument` without reading
and parsing their contents. If a stream or field exists in the file, the parser MUST either
fully parse it or return an error. Deferring parsing to "future milestones" by silently
suppressing errors is forbidden — it masks bugs and violates the fail-fast invariant.
`assert_all_consumed()` exists precisely to catch unhandled data; circumventing it defeats
the entire safety model.

**Use domain types from `altium-format-types`**: The `altium-format-types` crate defines typed
enums and structs for every Altium concept (`PcbObjectId`, `SchRecordType`, `Color`, `Coord`,
`UniqueId`, etc.) as well as named constants for format-level values (tag bytes, flag values,
type codes, masks, shifts). ALWAYS use these instead of raw primitives:
- Struct fields: `PcbObjectId` not `u8`, `SchRecordType` not `i32`, `Coord` not `i32`, etc.
- Constants: `INSTRUCTION_BINARY` not `0xD0`, `BLOCK_SIZE_MASK` not `0x00FF_FFFF`, etc.
- If a type or constant doesn't exist yet, add it to `altium-format-types` before using it.
  Types go in the appropriate module (`pcb.rs`, `sch.rs`, etc.); constants go in
  `crates/altium-format-types/src/constants/`. Make sure to check the constant you add against the decompiled code (Delphi or C# depending on the constant, but most should be in the already decompiled C# code)

NEVER use raw primitive integers where a domain type exists (e.g., use `Coord` not `i32`,
`SchRecordType` not `i32`, `PcbObjectId` not `u8`). If a type doesn't already exist in
`altium-format-types`, add one (discuss it with the user first).

**Strings:** Rust `String` (UTF-8) is the correct in-memory representation for decoded text.
Altium files use multiple encodings (Windows-1252 for parameter strings, UTF-8 via `%UTF8%`
prefix, UTF-16LE in pin sidecars and WideStrings). The encoding is a property of the
*serialization context*, not the string type — Altium's own C# code uses plain .NET `string`.
All encoding/decoding MUST happen at parse boundaries with strict error checking:
- Windows-1252: `encoding_rs::WINDOWS_1252.decode()` (all 256 byte values are valid, cannot error)
- UTF-16LE: `encoding_rs::UTF_16LE.decode()` — MUST check `had_errors` flag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akiselev/altium-cli](https://github.com/akiselev/altium-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
