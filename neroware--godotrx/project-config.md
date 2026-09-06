---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

GodotRx (GDRx) is a GDScript port of ReactiveX (ported from RxPY) for Godot Engine 4. It is a Godot editor plugin/addon, not a standalone application — the entire library lives under `addons/reactivex/`, and the rest of the repo (`project.godot`, `runtests.tscn`, `icon.png`) exists only to host the addon for local development and testing.

Since this is a near-direct port of RxPY, when the intent of an operator or scheduler is unclear, the [RxPY source/docs](https://rxpy.readthedocs.io/en/latest/) are the reference implementation to compare against.

## Running tests

There is no CLI test runner or CI config in this repo — tests run inside the Godot editor/engine itself:

1. Open the project in Godot 4.7 (`project.godot` sets `run/main_scene` to `res://runtests.tscn`).
2. Run the project (F5) or run `runtests.tscn` directly. This instantiates `__GDRx_TestRunner__` (`addons/reactivex/testing/testrunner.gd`), which recursively walks `addons/reactivex/testing/tests/` and executes every `*.test.gd` file, then quits.
3. Headless equivalent (if a `godot`/`godot4` binary is available): `godot --headless --path . res://runtests.tscn`.

Results are printed to the console per test method (`PASS`/`FAIL`) with a summary count — there is no separate assertion framework or exit-code-based failure signal beyond reading that output.

### Writing a test

Test files live in `addons/reactivex/testing/tests/` and must be named `*.test.gd`. They extend `__GDRx_Test__` (`addons/reactivex/testing/basetest.gd`). Every method named `test_*` is auto-discovered and run; it must return `bool` (`true` = failed, `false` = passed — this is inverted from typical assert conventions, matching `_compare_sequence`'s "difference found" semantics).

The core testing primitive is `compare(observable, expected_sequence)`, which subscribes, collects the full emission history, and diffs it against an expected array. Use the helpers `Err("ErrorTypeName")` and `Comp()` to represent an expected `on_error`/`on_completed` terminal event in that expected sequence:

```gdscript
extends __GDRx_Test__
const TEST_UNIT_NAME = "BASICS"

func test_rx_map() -> bool:
    var observable = GDRx.of([1, 2, 3, 4])
    var mapped = observable.map(func(x): return x * 2)
    return await compare(mapped, [2, 4, 6, 8, Comp()])
```

To add a new test *file*, just drop it in `testing/tests/` (optionally in a subdirectory — the runner recurses) — no registration step needed.

## Architecture

### Entry point: the `GDRx` autoload singleton

Everything is accessed through the `GDRx` singleton (`addons/reactivex/__gdrxsingleton__.gd`, autoloaded per `project.godot`'s `[autoload]` section and `plugin.gd`). It's a facade that aggregates several sub-modules as typed member fields rather than one monolithic class:

- `GDRx.obs` (`__gdrxobs__.gd`) — Observable *constructors* (`from_array`, `interval`, `zip`, `merge`, `timer`, ...). Each constructor is implemented as a `static func ..._()` in its own file under `observable/`, and `__gdrxobs__.gd` is a thin dispatch layer that `load()`s each file and forwards to it.
- `GDRx.op` (`__gdrxop__.gd`) — pipeable *operators* (`map_`, `filter_`, `take_`, ...), implemented one-per-file under `operators/` (and `operators/connectable/` for multicast-related ones) as `static func <name>_(...)  -> Callable`, following the same load-and-dispatch pattern. `GDRx.pipe` (`pipe.gd`) provides `compose`/`compose1..6` helpers for chaining operator Callables outside of `Observable.pipe(...)`.
- `GDRx.gd` (`engine/__gdrxengine__.gd`) — Godot-specific bridge: wraps Godot `Signal`s, coroutines, `_process`/`_physics_process`/input events, compute shaders, and `HTTPRequest` as Observables. The singleton exposes convenience wrappers on top of this (`from_signal`, `from_coroutine`, `on_mouse_down`, `on_key_just_pressed`, `on_idle_frame`, etc.).
- `GDRx.heap` / `GDRx.basic` / `GDRx.concur` / `GDRx.util` — internal building blocks (priority queue, identity/noop/equality helpers, threading primitives, iterables) sourced from `__gdrxinit__.gd`, mirrored under `internal/`.
- `GDRx.THREAD_MANAGER` / scheduler singletons (`ImmediateScheduler_`, `SceneTreeTimeoutScheduler_`, `ThreadedTimeoutScheduler_`, `NewThreadScheduler_`, `GodotSignalScheduler_`) — these are documented as "do NOT access directly"; go through the `GDRx.*` constructor/operator functions or explicit `Scheduler` classes instead.

New observable constructors and operators should follow this same split: implementation as a standalone `static func ..._()` file in `observable/` or `operators/`, then two thin forwarding wrappers added — one in `__gdrxobs__.gd`/`__gdrxop__.gd`, one in `__gdrxsingleton__.gd` (or `engine/__gdrxengine__.gd` for Godot-specific ones).

### Core abstractions (`abc/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Neroware/GodotRx](https://github.com/Neroware/GodotRx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
