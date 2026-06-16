---
trigger: always_on
description: Guidance for coding agents working in this C/libuv monoblok tree.
---

# AGENTS.md

Guidance for coding agents working in this C/libuv monoblok tree.

## Shape

Monoblok is a compact C17 NATS-like daemon with a patchbay routing DSL,
optional LVC, snapshots, JSON helpers, NATS bridge/core import, and
consumer-only JetStream import through vendored `nats.c`.

Keep the layout shallow:

- `src/`: core daemon pieces (`array`, `buf`, `fs`, `proto`, `router`,
  `slice`, `snapshot`, `bridge`, `importer`, `jetstream`, `nats_common`,
  `main`).
- `src/server/`: libuv listener, connection lifetime, read/write callbacks.
- `src/patchbay/`: arenas, S-expression/JSON/YAML adapters, program loading,
  evaluator dispatch, form fragments, validation, soundcheck, and dump tooling.
- `test/`: unit tests, NATS fakes, CLI checks, smoke/soundcheck scripts, and
  load/soak checks.
- `examples/`: runnable patchbay examples and demo configs.
- `advanced-examples/`: larger integration examples and producer/pump scripts.
- `docs/`: user-facing overview, patchbay docs, and documentation images.
- `design/`: design notes and proposals that are not necessarily product docs.
- `scripts/`: benchmark, smoke, packaging, install, service, and release helpers.
- `seccomp/`: container seccomp profile.
- `.github/workflows/`: CI and release automation.
- `.vscode/`: shared workspace editor settings.
- `bench-results/`: checked-in benchmark result notes.
- `vendor/libuv/`: vendored libuv source.
- `vendor/nats.c/`: vendored NATS C client.
- `vendor/yyjson/`: pruned yyjson source, license, and readme.
- Root files: CMake entrypoint, Dockerfile, readmes/agent notes, format/git
  config, and the default `patchbay.edn`.
- `build*/` and `dist/`: local/generated build and package output, not source
  layout.

Prefer local, explicit C over frameworks or abstraction layers. The point of
this branch is proving plain C can stay readable while staying fast.

## Project taste

Monoblok is close in spirit to the Redis Manifesto: a small daemon exposing a
clear data/protocol model, with memory-first predictable behavior and complexity
kept visible instead of hidden behind opaque layers.

- Treat the protocol, router, LVC, snapshots, and patchbay as concrete data
  structures with visible cost. APIs should make the model easier to reason
  about, not pretend hard tradeoffs disappeared.
- Prefer in-memory state and bounded, predictable hot paths. Disk is for
  snapshots and warm start, not a second storage engine unless the project
  explicitly changes shape.
- Say no to features that require broad abstraction, background magic,
  publish-time allocation, or large dependency surfaces for a marginal win.
- Dependencies are acceptable when they are self-contained, audited, and keep
  the main story smaller; vendored code should remain an island.
- Code should be pleasant to read because ownership, lifetimes, and performance
  consequences are obvious. Favor direct, local C over clever generality.

## Safety and correctness

Monoblok tends not to live directly on the public internet, but correctness
still matters. Bad input, accidental misuse, and future deployment changes can
find the same bugs as a hostile client.

- Be mindful of buffer overruns, integer overflow, unterminated data, borrowed
  slice lifetimes, and unchecked lengths whether the trigger looks nefarious or
  ordinary.
- Favor explicit logic over clever logic. Do not overcomplicate control flow,
  lifetime rules, or error handling for a marginal reduction in lines.
- Write simple, boring, descriptive C that a human can reason about under
  pressure. Prefer names that describe the domain object or state transition.
- Do not over-abstract. Keep helpers local until there is a real repeated
  domain concept.
- Do abstract common domain-specific operations behind project helpers with
  `mb_` names, such as file-open/read helpers. Memory helpers should make
  ownership obvious with `mb_<thing>_alloc` and `mb_<thing>_free` names.
- Keep load-bearing project structs briefly commented with their role,
  ownership, or lifetime. Do not comment for the sake of commentary; add
  comments when code is fiddly, protocol behavior is non-obvious, or an
  invariant would otherwise be easy to miss.
- Be open about repetitive work. Load tests, soak tests, and smoke tests should
  be automated where possible; editing repetitive comments by hand is often not
  productive.

## Build and test

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
ctest --test-dir build --output-on-failure
cmake --build build --target smoke
cmake --build build --target soundcheck
scripts/bridge-smoke.sh
scripts/import-smoke.sh
scripts/jetstream-smoke.sh
```

Sanitizer pass:

```sh
cmake -S . -B build-asan -DMB_ASAN=ON
cmake --build build-asan
ctest --test-dir build-asan --output-on-failure
```

Useful targets:

```sh
cmake --build build --target monoblok
cmake --build build --target bench-patchbay
cmake --build build --target pb-dump
```

## Core invariants

- The server runs on one libuv loop thread. Do not add hot-path mutexes or
  atomics unless the threading model changes.
- Keep one guarded `uv_write_t` per connection. Queue bytes in buffers, swap
  into `in_flight`, and resubmit only after completion.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lexvicacom/monoblok](https://github.com/lexvicacom/monoblok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
