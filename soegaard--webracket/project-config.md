---
trigger: always_on
description: **Project**: WebRacket — A Racket-to-WebAssembly Compiler and Runtime
---

# AGENTS.md

**Project**: WebRacket — A Racket-to-WebAssembly Compiler and Runtime  
**Purpose**: Implement the core Racket runtime and compiler targeting Wasm-GC using accurate value representation and Racket-compatible semantics.

---

## 🧠 Agent Overview

Codex agents working on this project must understand the **data
model**, **value tagging**, **type system**, and **semantic
goals**. Most work involves **emitting validated WebAssembly GC
code**, **writing runtime support functions**, and **implementing core
Racket primitives** faithfully.

### Bug-Finding Policy

- The MiniScheme interpreter is used to stress test WebRacket.
- When an error is found in WebRacket, do not code around the problem in examples.
- Instead, create a minimal reproducible WebRacket program and work on fixing WebRacket itself.
- Don’t work around compiler/runtime bugs in examples; make a minimal repro and fix core code.
- For compiler pass debugging, use `--dump-passes` and `--dump-passes-limit`; prefer `--no-stdlib` when possible for smaller dumps.

### Build and Publish Workflow

- Unless otherwise stated, assume screenshots are from content served from `local/`.
- For visual parity/debugging work, prefer Playwright-based computed-style comparisons over screenshot-only comparison when Playwright is available.
- Use screenshots as a fallback or supporting aid, not the primary source of truth, when computed-style inspection is possible.
- During development, build scripts must write to `local/` only.
- Do not write to `public/` from build scripts.
- `publish.sh` is the only script that copies from `local/` to `public/`.
- When rendering Scribble documentation locally, use
  `raco scribble --htmls --dest html scribblings/webracket.scrbl`
  so generated docs go into `html/`, which is ignored by git.

---

## Implementing new WebRacket primitives

The following steps are needed to implement a WebRacket primitive.

- Implement the primitive in `runtime-wasm.rkt`. 
- Add it to the list of primitives in `compiler.rkt`.
- If needed add a clause to the inlining in `compiler.rkt`
  (needed when the function doesn't take a fixed number of arguments)
- Export the primitive in `primitives.rkt`.
- If the WebRacket primitive isn't a primitive in Racket,
  then add a "dummy" implementation at the bottom of `primitives.rkt`.
  (this way a program written in `#lang webracket` works the same
   as if the program were compiled with webracket.
- Add test cases to `test/test-basics.rkt`. 
  Make sure to put the test cases in the correct section.
  (consult `docs/` to see the correct section)
- When asked to implement a WebRacket primitive, which is also
  in Racket, consult the documentation in `docs/` to get a 
  documentation for the function.
- If the WebRacket function has restrictions or behaves differently
  from the Racket one, make a comment about it in `runtime-wasm.rkt`
- If a parameter of a function is optional, mention it in an inline comment.
  Also, mention the default value.
- Use inline comments for the type of each parameter.

---

## 🔧 Core Responsibilities

| Agent Name         | Responsibility Description |
|--------------------|----------------------------|
| **type-checker**   | Validates types and enforces that functions only call `_checked` versions after verifying fixnum, character, string, etc. tags. Uses `ref.test` + `ref.cast`. |
| **formatter**      | Builds `display` and `write` routines for Racket values. Uses growable arrays/strings to avoid O(n²) behavior. |
| **utf8-agent**     | Maintains line/column/position tracking during UTF-8 output via `StringPort`. Handles CR, LF, CRLF, tab, and multibyte sequences. |
| **fasl-encoder**   | Encodes Racket values to FASL format. No graph sharing. Writes to a `GrowableBytes` buffer. Uses Racket’s FASL tag format. |
| **printer**        | Implements `format/display`, `format/display:symbol`, etc., dispatching based on tagged types or heap tags. |
| **structure-agent**| Builds and supports `make-struct-type-descriptor`, guards, accessors, mutators. Uses `$Struct`, `$StructType`, `$Array`. |
| **value-agent**    | Encodes immediate values: fixnums, characters, booleans. Knows tagging layout and validates properly. |
| **hash-agent**     | Supports mutable hash tables with open addressing. Uses `(ref $Array)` of alternating keys/values. |
| **closure-agent**  | Implements closures as `(ref $Closure)` with `$ClosCode` and `$Free` array. Follows Racket's argument vector model. |
| **symbol-agent**   | Manages symbol interning and gensyms. Symbols store interned names in `$String` form. |
| **growable-agent** | Constructs growable string/byte/int builders using `Growable*` types and converts to final immutable arrays. |

---

## 🧱 Representation

### Heap and Array

(type $Array    (array (mut (ref eq))))
(type $I32Array (array (mut i32)))
(type $I8Array  (array (mut i8)))

### Boxed Types

(type $Bytes (sub $Heap
  (struct (field $hash (mut i32))
          (field $immutable i32)
          (field $bs (mut (ref $I8Array))))))

(type $String (sub $Heap
  (struct (field $hash (mut i32))
          (field $immutable i32)
          (field $codepoints (mut (ref $I32Array))))))

(type $Pair (sub $Heap
  (struct (field $hash (mut i32))
          (field $a (mut (ref eq)))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soegaard/webracket](https://github.com/soegaard/webracket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
