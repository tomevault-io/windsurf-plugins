---
trigger: always_on
description: Code style rules — naming, guard clauses, dispatch, variable reassignment, mutation, strings, truncation, imports.
---

# Code Style
- Follow PEP 8 style guidelines (enforced by Ruff)
- Use descriptive variable names: `timeout_seconds` not `t`, `worker_count` not `n`
- Boolean variables and functions: prefix with `is_`, `has_`, `can_`, `should_`
- Keep functions focused and single-purpose (Single Responsibility Principle)
- Add comprehensive docstrings for all public APIs (Google style with Args/Returns/Raises)
- Include usage examples in docstrings for complex functionality

## Guard Clauses and Early Returns

Reduce nesting by checking preconditions first and returning/raising early. This keeps the happy path at low indentation and makes control flow obvious.

❌ Bad (deeply nested):
```python
def process_task(self, task: Task) -> Result:
    if task is not None:
        if task.is_valid():
            if self._is_running:
                return self._execute(task)
            else:
                raise WorkerStoppedError()
        else:
            raise InvalidTaskError()
    else:
        raise ValueError("task is None")
```

✅ Good (guard clauses):
```python
def process_task(self, task: Task) -> Result:
    if task is None:
        raise ValueError("task is None")
    if not task.is_valid():
        raise InvalidTaskError()
    if not self._is_running:
        raise WorkerStoppedError()
    return self._execute(task)
```

## Exhaustive Dispatch: `if-elif-else` with a Raising `else`

When a variable has a **closed set of valid values** and you are dispatching behavior based on it, always use a full `if-elif-else` ladder where the `else` branch raises an exception. This makes two things immediately clear to the reader: (a) exactly which values are handled, and (b) that no value falls through silently.

Contrast this with a **feature flag** (a boolean or optional condition that enables/disables a behavior). Feature flags correctly use a bare `if` with no `else`, because the "else" case is "do nothing, proceed normally" — not an error.

**Closed-set dispatch — ALWAYS use `if-elif-else` with raising `else`:**

❌ Bad (validate-then-dispatch — reader must remember the set of valid values):
```python
algorithm: str = config.get("load_balancing", "round_robin")
if algorithm not in ("round_robin", "least_loaded", "random"):
    raise ValueError(f"Unknown algorithm={algorithm!r}")

if algorithm == "round_robin":
    return RoundRobinBalancer(...)
if algorithm == "least_loaded":
    return LeastLoadedBalancer(...)
return RandomBalancer(...)  # ← reader must infer this is the "random" case
```

✅ Good (exhaustive dispatch — every case is explicit, invalid values caught):
```python
if algorithm == "round_robin":
    return RoundRobinBalancer(...)
elif algorithm == "least_loaded":
    return LeastLoadedBalancer(...)
elif algorithm == "random":
    return RandomBalancer(...)
else:
    raise ValueError(
        f"Unknown load_balancing algorithm={algorithm!r}. "
        f"Must be 'round_robin', 'least_loaded', or 'random'."
    )
```

**Why the `else` matters:** Without it, adding a fourth algorithm value later silently falls through to the last branch. With `else: raise`, the new value is caught immediately. This is the textual equivalent of a `match/case` with exhaustiveness checking.

**Feature flag — bare `if` is correct:**

✅ Good (optional behavior, not a dispatch):
```python
if self.enable_metrics:
    self._metrics_collector.record(elapsed_ms)
```

Here, the `if` block enables an optional feature. There is no `else` because the alternative is "don't record metrics" — that is a valid default, not an error. An `else` branch would be empty noise.

**Decision rule:** Ask "is the `else` case an error, or is it 'do nothing'?"
- **Error** (closed set dispatch) → `if-elif-else` with raising `else`
- **Do nothing** (feature flag) → bare `if`, no `else`

## Naming Conventions

- `snake_case` for functions, methods, variables, modules
- `CamelCase` for classes
- `UPPER_SNAKE_CASE` for module-level constants
- Prefix private attributes with `_`: `self._internal_state`
- Name by intent, not implementation: `failed_futures` not `bad_list`
- Include units where ambiguous: `timeout_seconds`, `max_retries`, `delay_ms`
- Avoid abbreviations: `configuration` not `cfg`, `response` not `resp`, `message` not `msg`

## Never Rename Variables Just to Shorten Them (Namespace Pollution)

**Never create a new local variable that is just a shorter or abbreviated alias for an existing parameter, attribute, or variable.** This is namespace pollution: it forces the reader to hold two names for the same value in their head, and the shorter name invariably loses the meaning that the original name carried.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
