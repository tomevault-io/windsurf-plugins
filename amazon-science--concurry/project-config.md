---
trigger: always_on
description: Runtime type enforcement with @validate — when to use, performance and serialization exemptions.
---

# Runtime Type Enforcement with `@validate` (CRITICAL)

## What `@validate` Does and Why It Matters

`morphic.validate` is a decorator built on Pydantic's `validate_call` that brings full Pydantic validation to regular functions and methods. When you decorate a function with `@validate`, every call validates and coerces its arguments against the declared type hints at runtime — not just at type-check time, not just in tests, but on every single invocation.

This is the bridge between "type hints as documentation" and "type hints as enforcement." Without `@validate`, a function annotated `def submit(task: Task)` will happily accept a string, a number, or `None` at runtime — Python ignores annotations by default. With `@validate`, the same function rejects invalid inputs immediately with a clear `ValidationError`, and automatically coerces compatible types (e.g., a dict into a `Task` object via Pydantic's model coercion).

The interaction with the Strict `Any` Ban is direct: `@validate` can only enforce types that are declared. If a parameter is typed `Any`, `@validate` passes it through without checking. Every `Any` in a `@validate`-decorated function is a parameter that bypasses runtime validation entirely. The two rules reinforce each other: the `Any` ban ensures types are declared precisely, and `@validate` ensures those precise declarations are enforced at runtime.

For LLM coding agents, `@validate` is a critical safety net. When an agent generates a call like `worker.submit(task=some_variable)`, the agent may have constructed `some_variable` incorrectly — wrong type, wrong structure, missing fields. Without `@validate`, this error propagates silently. With `@validate`, the error is caught at the function boundary with a clear `ValidationError` at the call site.

`@validate` works with Morphic `Typed` classes, `Typed + Registry` subclasses, standalone functions, and class methods. Dict arguments are automatically coerced into `Typed` instances. String arguments are coerced into ints, floats, and bools. `AutoEnum` values are resolved by fuzzy matching. This coercion means `@validate` is not just a gate — it is a normalizer that converts loosely-typed data into strongly-typed objects at the function boundary.

The latency cost is real but small: a few hundred microseconds per call. For Concurry's user-facing API (`.options()`, `.init()`, `gather()`, `wait()` called with a list) this is invisible — the actual work (thread/process/Ray dispatch) dwarfs the validation cost by orders of magnitude. The only places where `@validate` must be omitted are **tight inner loops in concurrency primitives** where microseconds compound into seconds.

### When to Use `@validate` in Concurry

**Default: use `@validate` on every public function and method.** The decorator should be present unless a specific exception applies.

| Category | Use `@validate`? | Rationale |
|---|---|---|
| **All public module-level functions** (`wait`, `gather`, `wrap_future`) | **YES** | User-facing entry points. |
| **All public methods of user-facing classes** (`Worker.options`, `LimitSet.acquire`) | **YES** | `Worker.options()` already uses `@validate`. Extend to all public methods. |
| **Factory functions** | **YES** | Construct objects from user config; coercion is essential. |
| **Private methods** (`_create_retry_configs`, `_dispatch_to_worker`) | **No** (type hints required, `@validate` optional) | Called from validated public methods. Double-validating wastes cycles. |
| **Morphic lifecycle hooks** (`pre_initialize`, `post_initialize`, `pre_validate`) | **No** | The Morphic framework calls these with already-validated data. |
| **Pydantic `@field_validator` methods** | **No** | Already part of Pydantic's validation pipeline. |
| **`AutoEnum` methods, `__getattr__`, `__getitem__`** | **No** | Input types guaranteed by the Python language or Morphic framework. |
| **Hot-path concurrency primitives** (see below) | **No** (with documented justification) | Called thousands/millions of times per second. |
| **Worker methods serialized across Ray** (see below) | **Case-by-case** | `@validate` closure may not be picklable. |

### Performance Exemption (CRITICAL for Concurry)

Concurry's core primitives are called in tight loops at very high frequency. The following components are **explicitly exempt** from `@validate` because the validation overhead compounds into visible latency:

| Component | Call frequency | Why exempt |
|---|---|---|
| `BaseFuture.result()`, `.done()`, `.cancel()`, `.exception()` | Once per future in `wait()`/`gather()` loops (50,000+/sec) | 200µs × 50k = 10s added to a 2s gather |
| `async_wait()`, `async_gather()` inner loops | Same as above | Async hot path |
| `Acquisition.__init__()`, `.update()`, `.release()` | Once per limit acquisition per task submission | Thousands/sec in rate-limited workers |
| `LimitSet` internal synchronization methods | Called under lock on every task dispatch | Microseconds matter under contention |

When `@validate` is omitted from a public function for performance, the omission **MUST** be documented with a comment:

```python
# @validate omitted: hot path — called once per future in wait()/gather() loops.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
