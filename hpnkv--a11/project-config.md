---
trigger: always_on
description: - `a11/` defines the current Python API and behavioural contract. `cpp/` is
---

# A11 Engineering Guide

## Source of truth and layout

- `a11/` defines the current Python API and behavioural contract. `cpp/` is
  its native implementation and should converge on the same semantics.
  `actionengine/` is historical reference material, not the source of truth.
- Keep independently linkable C++ components (`core`, `data`, `concurrency`,
  `stores`, `net`, `nodes`, `actions`, and `service`). Each `cpp/a11/<part>/`
  owns its source list in its local `CMakeLists.txt`; the target and dependency
  graph remain visible in `cpp/CMakeLists.txt`.
- Concurrency types live directly in `namespace a11`, even though their files
  remain under `a11/concurrency/`. Do not recreate `a11::concurrency`.
- Byte-level primitives that several unrelated components need are **header-only
  inline** functions, so sharing them costs no link dependency: `a11/utf8.h`
  (`IsContinuation`, `SequenceWidth`, `Utf16Units`, `IsValid`) and
  `a11/percent.h` (`HexDigit`, `Decode`, `DecodeStrict`, `Encode`). That is what
  lets `a11::flow_lang`, which links nothing but Abseil and nlohmann, use the
  same UTF-8 rules as the serializer. Reach for these rather than open-coding a
  lead-byte ladder or a `%xx` loop, and add new ones to
  `cpp/tests/header_canary.cc`. The strict UTF-8 validator has one home,
  `a11::IsValidUtf8` in `a11/json_codec.h`; `FindUnencodableString` beside it is
  iterative on purpose, because fiber stacks are small and fixed.
- Public stateful Python runtime types reuse the bound native class objects;
  do not add shadow Python implementations or facade subclasses for concrete
  native types. Attach thin, asynchronous, idiomatic protocols to the bound
  classes, while retaining deliberate virtual adapters such as
  `LocalChunkStore` where Python subclass overrides must cross into C++.
- Pydantic-style validation, JSON, copy, and schema helpers augment native
  binary and schema values; they must not introduce a second public data model.
  Python serializers and deserializers continue to operate on those native
  values.
- A Python reimplementation of logic the native library already has needs a
  reason a caller can feel: pydantic and `IntEnum` ergonomics, the exception a
  Python developer expects, duck typing a C++ signature cannot express. Parity
  is not a reason. Where the only difference is language, delegate to the
  binding and delete the copy — a second implementation is one that falls
  behind, and the fix is to remove it rather than to test it. This does not
  license collapsing the per-language *data tables* below (serial tags, status
  chunks): those exist once per language because each language needs its own
  literals, and `testdata/` pins them to each other.
- Flow is the language whose programs are compositions of actions that are
  themselves actions. **The language lives in `cpp/a11/flow/`**: the lexer, the
  highlighter, the parser, the resolver, the inspector, the formatter, the
  completion **and the runtime** are native, and every surface is a frontend over
  them — the `a11 flow` CLI, the `a11._native.flow` bindings, the standalone
  `a11-flow` binary, and the IntelliJ plugin. Nothing about the language is
  implemented in Python: `a11/flow/plan.py` and `a11/flow/runtime.py` are glue over
  `a11._native.flow`, and `flow.loads` is the strict door onto it.
    - The point of the move is that there is **one** implementation of each language
      judgement. There is no lexer, parser, resolver, inspector or word list in
      Kotlin, Python or a grammar file any more, and there must not be one again: a
      second copy is a copy that falls behind, and the fix is to delete it rather
      than test it. `a11/flow/tests/test_editor_support.py` now holds the *absence*
      of those copies.
    - The language tooling is `a11::flow_lang`, and it links **nothing but Abseil and
      nlohmann**. No sockets, no nodes, no OpenSSL. That is what lets it ship as
      `a11-flow` (a few megabytes, `A11_BUILD_FLOW_TOOL`) on platforms where the full
      runtime is not built, so keep it that way. The executing half is the separate
      `a11::flow_runtime` (`values.cc`, `runtime.cc`), which does link the node and
      action layers — and `a11_flow_test`'s link line is the proof the language does
      not.
    - The runtime is **fibers over one monitor**: `thread::` primitives only (never
      `std::mutex`), one lock and one condition variable per run, and blocking work
      done with the lock released. That is what makes giving up on a run possible at
      all — a pump waiting for a reader that will never come has to be woken. Every
      fiber gets a stack an interpreter frame chain fits in, because any of them can
      reach the `HostBridge`.
    - `HostBridge` is the three questions only the host can answer: coerce a value
      into a registered type, read one out of a chunk, write one into a chunk. The
      Python bindings answer them against the Python registry -- which is where a
      pydantic model actually lives -- and the standalone tool answers them with the
      C++ registry. A `Value` the language cannot take apart is carried as an opaque
      host object; the language only ever renders, indexes, compares or truth-tests
      one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hpnkv/a11](https://github.com/hpnkv/a11) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
