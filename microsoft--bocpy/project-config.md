---
trigger: always_on
description: bocpy is a Python library implementing **Behavior-Oriented Concurrency (BOC)** —
---

# bocpy — Copilot Instructions

## What bocpy Is

bocpy is a Python library implementing **Behavior-Oriented Concurrency (BOC)** —
a concurrency paradigm that eliminates data races and deadlocks by construction.
Instead of locks and shared mutable state, programmers wrap data in **cowns**
(concurrently-owned objects) and schedule **behaviors** that execute once all
required cowns are available. The scheduler acquires cowns in a deterministic
order (two-phase locking over sorted cown IDs), guaranteeing deadlock freedom.

Workers run in **Python sub-interpreters** — on Python 3.12+ these are truly
parallel (per-interpreter GIL). An Erlang-style **message queue** (`send`/`receive`)
implemented as a lock-free multi-producer single-consumer (MPSC) ring buffer in
C provides cross-interpreter communication.

## Architecture

```
User code (@when)
       │
       ▼
 Python layer (behaviors.py)
   ├── @when decorator: captures closure vars, schedules behavior
   │     directly from the caller's thread (no central queue hop)
   ├── Behaviors: runtime lifecycle, worker pool, terminator
   │     (no scheduler thread; scheduling and release run on the
   │      threads that need them — caller and worker)
   ├── Cown: typed data wrapper with acquire/release semantics
   └── Noticeboard: notice_write/update/delete/read, noticeboard()
         (global key-value store, snapshot-per-behavior; mutators
          are serialized through one dedicated noticeboard thread)
       │
       ▼
 Transpiler (transpiler.py)
   └── AST transforms: extracts @when functions, rewrites closures
       as explicit parameters, exports module for worker import
       │
       ▼
 Worker (worker.py)
   └── Sub-interpreter event loop: receives behavior capsules via
       boc_worker queue, executes them, then releases cowns and
       decrements the terminator on the worker thread itself
       │
       ▼
 C extensions
   ├── _core.c: CownCapsule, BehaviorCapsule, BOCBehavior +
   │            BOCRequest array (two-phase locking), C-level
   │            terminator, lock-free MPSC message queues (16
   │            queues, tag-based), global Noticeboard (mutex-
   │            protected, up to 64 entries with thread-local
   │            snapshot cache + monotonic version counter)
   └── _math.c: dense double-precision Matrix
```

### Key data flow

1. `@when(cown_a, cown_b)` → transpiler extracts the decorated function and its
   captured variables → exported as `__behavior__N` in a generated module.
2. `whencall` (caller's thread) increments the C terminator and calls
   `_core.behavior_schedule`, which performs two-phase locking (2PL) over
   the sorted cown IDs in C, releasing the GIL across the lock-free link
   loops.
3. When all cowns are acquired, `behavior_resolve_one` enqueues the
   `BehaviorCapsule` directly to the `boc_worker` queue — no central
   scheduler hop.
4. A worker pops the capsule, executes `__behavior__N` with exclusive access
   to the cowns, then on the same worker thread calls
   `behavior_release_all` (MCS unlink + handoff to the next behavior
   waiting on each cown) and `terminator_dec`.
5. Releasing a cown may resolve the next waiting behavior, which is
   dispatched directly to a worker without touching any central queue.
   The result is stored in the `Cown` returned by `@when`.
6. `wait()` blocks on `terminator_wait` until the C-level count reaches
   zero; `stop()` then drains the workers and the noticeboard thread.

## Public API

| Symbol | Purpose |
|--------|---------|
| `Cown[T]` | Typed wrapper for concurrently-owned data (with `.value` and `.exception` properties) |
| `@when(*cowns)` | Schedule a behavior with exclusive access to the listed cowns |
| `whencall(thunk, args, captures)` | Lower-level form of `@when` used by the transpiler; schedules a named thunk against cowns and capture values |
| `send(tag, contents)` | Send a message to a tag (lock-free) |
| `receive(tags, timeout, after)` | Selective receive; blocks or times out |
| `drain(tags)` | Clear all queued messages for the given tag(s) |
| `set_tags(tags)` | Pre-assign tags to queues; clears all messages |
| `TIMEOUT` | Sentinel returned by `receive` on timeout |
| `noticeboard()` | Read a per-behavior cached snapshot of the global key-value store |
| `notice_read(key, default)` | Convenience: read a single key from the snapshot |
| `notice_write(key, value)` | Non-blocking write to the noticeboard |
| `notice_update(key, fn, default)` | Atomic read-modify-write; returning `REMOVED` deletes the entry |
| `notice_delete(key)` | Non-blocking delete of a single noticeboard entry |
| `REMOVED` | Sentinel returned by a `notice_update` fn to delete the entry |
| `wait(timeout)` | Block until all behaviors complete; stops the runtime |
| `start(workers, export_dir, module)` | Manually start the runtime (auto-called on first `@when`) |
| `Matrix` | Dense 2D matrix of doubles with C-backed arithmetic |
| `WORKER_COUNT` | Default worker count (CPU count − 1) |

## Project Layout

| Path | Contents |
|------|----------|
| `src/bocpy/__init__.py` | Public re-exports |
| `src/bocpy/__init__.pyi` | Type stubs (Sphinx-style docstrings) |
| `src/bocpy/behaviors.py` | `Cown`, `@when`, `Behaviors` scheduler, runtime lifecycle |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/bocpy](https://github.com/microsoft/bocpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
