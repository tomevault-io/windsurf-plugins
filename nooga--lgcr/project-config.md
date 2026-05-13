---
trigger: always_on
description: lgcr is a container runtime written in let-go (Clojure dialect → Go bytecode).
---

# lgcr Development Guide

## Overview

lgcr is a container runtime written in let-go (Clojure dialect → Go bytecode).

- `container.lg` — runtime (pull, run, shim, init, ps, logs, stop, kill, rm, start, inspect)
- `lib.lg` — pure helpers (no filesystem / syscall / env access) used by container.lg and unit tests
- Depends on let-go's `syscall` namespace (`../let-go/pkg/rt/syscall_linux.go`)

## Build

```bash
./bundle.sh              # builds host + linux lg, cross-bundles to ./lgcr
```

`bundle.sh` produces `./lgcr`, a standalone Linux arm64 binary with the let-go
VM + compiled container.lg bytecode embedded. `.lg-host` (host build) and
`.lg-linux` (target) are side artifacts used for bundling and unit tests.

## Testing — **required for every new feature**

Any new feature, subcommand, or helper **must** come with test coverage in the
same patch. No exceptions. Two suites:

### Unit tests — `tests/lib_test.lg`

Pure helpers in `lib.lg` are tested here using let-go's built-in `test` ns
(`deftest`, `is`, `testing`). Add new tests next to the existing ones — one
`deftest` per function, covering the happy path and edge cases.

```clojure
(deftest new-function-description
  (is (= expected (lib/new-function input))))
```

Run them standalone on the host (fast, no Lima):
```bash
./.lg-host tests/lib_test.lg
```

### E2E tests — `tests/e2e.sh`

New subcommands and behaviors that touch the real runtime go here. The script
runs inside Lima against the actual `./lgcr` binary. Use the provided helpers:

```bash
section "my-new-feature behaves correctly"
OUT=$("$LGCR" my-new-command "$IMG" args 2>&1)
expect_contains "$OUT" "expected string" "description"
expect_eq "$(json_field "$STATE_FILE" some-field)" "some-value"
```

### Running the whole suite

```bash
./tests/run.sh           # bundle + unit tests on host + e2e in Lima
```

This is the gate — run it before committing anything non-trivial. All tests
must pass.

### When to put something in lib.lg vs container.lg

- **lib.lg**: string/collection manipulation, parsing, formatting, anything
  that takes values in and returns values out. No `os/*`, `syscall/*`, `slurp`,
  `spit`, `sh!`, or state reads.
- **container.lg**: everything that touches the filesystem, spawns processes,
  reads env, or manipulates the state store.

If a function in container.lg starts growing a lot of pure logic, extract the
pure part into lib.lg and add a unit test.

## Code Style / Idiomatic let-go

Write let-go as idiomatic Clojure unless a documented let-go limitation says
otherwise.

- Use proper namespace forms. Prefer `:as` aliases for required namespaces
  (`[string :as str]`, `[lib :as lib]`, etc.). Use `:refer` only for small,
  stable APIs where unqualified names improve readability, such as test macros.
- Prefer destructuring wherever it clarifies the data shape: function arguments,
  `let` bindings, `loop` bindings, and nested map/vector values.
- Use threading macros (`->`, `->>`, `cond->`, and related forms where
  supported) when they make transformation pipelines easier to read.
- Prefer small pure helper functions and data transformations over deeply nested
  inline logic.
- Use maps, vectors, keywords, and immutable updates idiomatically with `assoc`,
  `update`, `dissoc`, `select-keys`, `merge`, `into`, and explicit reducers.
- Keep side effects at the runtime boundary in `container.lg`; keep parsing,
  formatting, and collection logic in `lib.lg` with tests.
- Use qualified aliases consistently. Avoid mixing full namespace names and
  aliases in new code unless touching old code narrowly.
- Be careful with let-go-specific namespace behavior. In particular, `async`
  refers publics unqualified and shadows core names like `map`, `into`,
  `reduce`, `take`, and `merge`; avoid requiring it in namespaces that use
  those names unqualified unless the code is structured to account for it.

## Lima VM

The test VM is named `letgo`. If not present:
```bash
limactl start --name=letgo --vm-type=vz --mount-writable --cpus=2 --memory=2
```

Mounts the macOS filesystem read-write. Run commands via:
```bash
limactl shell letgo sudo <command>
```

## Project Structure

- `container.lg` — runtime
- `lib.lg` — pure helpers
- `bundle.sh` — cross-bundle host → linux/arm64
- `tests/lib_test.lg` — unit tests (pure helpers)
- `tests/e2e.sh` — e2e tests (real binary in Lima)
- `tests/run.sh` — top-level runner
- `ROADMAP.md` — milestones + completion status

## Key Design Decisions

- **Two-phase run**: host side prepares overlay + cgroups, then spawns
  `lgcr init <id>` with CLONE_NEW{NS,PID,UTS,IPC} via `syscall/spawn-async`.
  The init becomes PID 1 in the new PID namespace; it spawns the user
  command and stays alive as reaper + signal forwarder.
- **Per-container shim**: `run -d` spawns `lgcr shim <id>` detached
  (Setsid'd). Shim owns the container's lifecycle: does rootfs + cgroup
  setup, launches init, waitpids it, writes final state. No daemon.
- **Overlay with cp fallback**: overlay mount fails in nested containers
  (podman), so we fall back to `cp -a`. On bare Linux (Lima) overlay works.
- **OCI pull**: real registry client with token auth, manifest list
  resolution, streaming layer download. Saves reduced image config to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nooga/lgcr](https://github.com/nooga/lgcr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
