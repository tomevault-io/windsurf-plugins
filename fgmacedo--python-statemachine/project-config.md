---
trigger: always_on
description: Python Finite State Machines made easy.
---

# python-statemachine

Python Finite State Machines made easy.

## Project overview

A library for building finite state machines in Python, with support for sync and async engines,
Django integration, diagram generation, and a flexible callback/listener system.

- **Source code:** `statemachine/`
- **Tests:** `tests/`
- **Documentation:** `docs/` (Sphinx + MyST Markdown, hosted on ReadTheDocs)

## Architecture

- `statemachine.py` — Core `StateMachine` and `StateChart` classes
- `factory.py` — `StateMachineMetaclass` handles class construction, state/transition validation
- `state.py` / `event.py` — Descriptor-based `State` and `Event` definitions
- `transition.py` / `transition_list.py` — Transition logic and composition (`|` operator)
- `callbacks.py` — Priority-based callback registry (`CallbackPriority`, `CallbackGroup`)
- `dispatcher.py` — Listener/observer pattern, `callable_method` wraps callables with signature adaptation
- `signature.py` — `SignatureAdapter` for dependency injection into callbacks
- `engines/base.py` — Shared engine logic (microstep, transition selection, error handling)
- `engines/sync.py`, `engines/async_.py` — Sync and async processing loops
- `registry.py` — Global state machine registry (used by `MachineMixin`)
- `mixins.py` — `MachineMixin` for domain model integration (e.g., Django models)
- `spec_parser.py` — Boolean expression parser for condition guards
- `contrib/diagram.py` — Diagram generation via pydot/Graphviz

## Processing model

The engine follows the SCXML run-to-completion (RTC) model with two processing levels:

- **Microstep**: atomic execution of one transition set (before → exit → on → enter → after).
- **Macrostep**: complete processing cycle for one external event; repeats microsteps until
  the machine reaches a **stable configuration** (no eventless transitions enabled, internal
  queue empty).

### Event queues

- `send()` → **external queue** (processed after current macrostep ends).
- `raise_()` → **internal queue** (processed within the current macrostep, before external events).

### Error handling (`catch_errors_as_events`)

- `StateChart` has `catch_errors_as_events=True` by default; `StateMachine` has `False`.
- Errors are caught at the **block level** (per onentry/onexit/transition `on` block), not per
  microstep. This means `after` callbacks still run even when an action raises — making
  `after_<event>()` a natural **finalize** hook (runs on both success and failure paths).
- `error.execution` is dispatched as an internal event; define transitions for it to handle
  errors within the statechart.
- Error during `error.execution` handling → ignored to prevent infinite loops.

#### `on_error` asymmetry: transition `on` vs onentry/onexit

Transition `on` content uses `on_error` **only for non-`error.execution` events**. During
`error.execution` processing, `on_error` is disabled for transition `on` content — errors
propagate to `microstep()` where `_send_error_execution` ignores them. This prevents infinite
loops in self-transition error handlers (e.g., `error_execution = s1.to(s1, on="handler")`
where `handler` raises). `onentry`/`onexit` blocks always use `on_error` regardless of the
current event.

### Eventless transitions

- Bare transition statements (not assigned to a variable) are **eventless** — they fire
  automatically when their guard condition is met.
- Assigned transitions (e.g., `go = s1.to(s2)`) create **named events**.
- `error_` prefix naming convention: `error_X` auto-registers both `error_X` and `error.X`
  event names (explicit `id=` takes precedence).

### Callback conventions

- Generic callbacks (always available): `prepare_event()`, `before_transition()`,
  `on_transition()`, `on_exit_state()`, `on_enter_state()`, `after_transition()`.
- Event-specific: `before_<event>()`, `on_<event>()`, `after_<event>()`.
- State-specific: `on_enter_<state>()`, `on_exit_<state>()`.
- `on_error_execution()` works via naming convention but **only** when a transition for
  `error.execution` is declared — it is NOT a generic callback.

### Thread safety

- The sync engine is **thread-safe**: multiple threads can send events to the same SM instance
  concurrently. The processing loop uses a `threading.Lock` so at most one thread executes
  transitions at a time. Event queues use `PriorityQueue` (stdlib, thread-safe).
- **Do not replace `PriorityQueue`** with non-thread-safe alternatives (e.g., `collections.deque`,
  plain `list`) — this would break concurrent access guarantees.
- Stress tests in `tests/test_threading.py::TestThreadSafety` exercise real contention with
  barriers and multiple sender threads. Any change to queue or locking internals must pass these.

### Invoke (`<invoke>`)

- `invoke.py` — `InvokeManager` on the engine manages the lifecycle: `mark_for_invoke()`,
  `cancel_for_state()`, `spawn_pending_sync/async()`, `send_to_child()`.
- `_cleanup_terminated()` only removes invocations that are both terminated **and** cancelled.
  A terminated-but-not-cancelled invocation means the handler's `run()` returned but the owning
  state is still active — it must stay in `_active` so `send_to_child()` can still route events.
- **Child machine constructor blocks** in the processing loop. Use a listener pattern (e.g.,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fgmacedo/python-statemachine](https://github.com/fgmacedo/python-statemachine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
