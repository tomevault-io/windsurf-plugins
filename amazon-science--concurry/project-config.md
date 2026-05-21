---
trigger: always_on
description: These architecture details are useful when designing, integrating or refactoring major components of concurry, when debugging challenging errors and race conditions, and when testing cross-feature functionality. Also refer to the respective files in docs/architecture/ for comprehensive details.
---

# Cursor Rules: Concurry Architecture

## Synchronization Architecture and Rules

**CRITICAL**: Follow these rules when working with `wait()`, `gather()`, or future implementations.

For comprehensive architecture details, see [docs/architecture/synchronization.md](../../docs/architecture/synchronization.md)

### BaseFuture Implementations

**Rule 1**: Each future type has a specific state management strategy:
- `SyncFuture`: Caches everything (immutable at creation)
- `ConcurrentFuture`: Pure delegation wrapper (NO caching of `_result`, `_exception`, `_done`, `_cancelled`)
- `AsyncioFuture`: Pure delegation wrapper (NO caching of `_result`, `_exception`, `_done`)
- `RayFuture`: Caches state after fetching (stores `_result`, `_exception`, `_done`, `_cancelled`)

**Rule 2**: NEVER set `_done=True` without fetching the result:
```python
# ❌ WRONG - RayFuture bug
def done(self) -> bool:
    ready, _ = ray.wait([self._object_ref], timeout=0)
    if len(ready) > 0:
        self._done = True  # ❌ BUG: _result is still None!
        return True

# ✅ CORRECT
def done(self) -> bool:
    if self._done:
        return True
    ready, _ = ray.wait([self._object_ref], timeout=0)
    return len(ready) > 0  # Don't set _done here
```

**Rule 3**: All futures must raise consistent exception types:
- Raise `concurrent.futures.CancelledError`, NOT `asyncio.CancelledError`
- Raise `TimeoutError`, NOT `ray.exceptions.GetTimeoutError`

**Rule 4**: Callbacks must receive the wrapper (BaseFuture), not the underlying future:
```python
# ✅ CORRECT
def add_done_callback(self, fn: Callable) -> None:
    self._future.add_done_callback(lambda _: fn(self))  # Pass self, not _
```

**Rule 5**: Always use `wrap_future()` when accepting external futures:
```python
# ✅ CORRECT
from concurry.core.future import wrap_future

futures_list = [wrap_future(f) for f in external_futures]
```

### wait() and gather() Functions

**Rule 6**: Cannot mix structure and variadic arguments:
```python
# ❌ WRONG
futures = [f1, f2, f3]
wait(futures, f4, f5)  # Raises ValueError

# ✅ CORRECT
wait([f1, f2, f3, f4, f5])  # Pass as list
wait(f1, f2, f3, f4, f5)     # Pass individually
```

**Rule 7**: Dict inputs must return dicts with preserved keys:
```python
tasks = {"task1": f1, "task2": f2}
results = gather(tasks)
# Must return: {"task1": r1, "task2": r2}
```

**Rule 8**: Always call `fut.result(timeout=0)` after `wait()` completes:
```python
# ✅ CORRECT
done, not_done = wait(futures_list, ...)
for fut in done:
    result = fut.result(timeout=0)  # timeout=0 is safe, future is done
```

**Rule 9**: Ray batch checking must use single `ray.wait()` call:
```python
# ✅ CORRECT - batch all Ray futures
ready, not_ready = ray.wait(
    ray_futures, 
    num_returns=len(ray_futures),  # Check all at once
    timeout=0
)
```

### Polling Strategies

**Rule 10**: All polling intervals must come from `global_config`:
```python
# ❌ WRONG
strategy = FixedPollingStrategy(interval=0.01)  # Hardcoded!

# ✅ CORRECT
from concurry.config import global_config
defaults = global_config.defaults
strategy = FixedPollingStrategy(interval=defaults.polling_fixed_interval)
```

**Rule 11**: New polling strategies must inherit from `BasePollingStrategy`:
```python
class CustomStrategy(BasePollingStrategy):
    aliases = ["custom", PollingAlgorithm.Custom]
    
    def get_next_interval(self) -> float: ...
    def record_completion(self) -> None: ...
    def record_no_completion(self) -> None: ...
    def reset(self) -> None: ...
```

### Extension Guidelines

**Adding New Future Type**:
1. Inherit from `BaseFuture` and implement all abstract methods
2. Define `__slots__` with minimal fields (delegate when possible)
3. Use `FUTURE_UUID_PREFIX` class variable
4. Update `wrap_future()` function to detect new type
5. Update `_check_futures_batch()` for batch optimization (if applicable)

**Adding New Polling Strategy**:
1. Inherit from `BasePollingStrategy` (Registry + MutableTyped)
2. Define `aliases` for factory creation
3. Add to `PollingAlgorithm` enum
4. Add configuration defaults to `GlobalDefaults`
5. Update strategy creation in `wait()` and `gather()`

## Core Design Principle: No Silent Fallbacks

**CRITICAL**: Concurry must **fail noisily** rather than silently degrade performance.

### The Rule

❌ **NEVER** auto-fallback to slower implementations when configured implementation fails
✅ **ALWAYS** fail loudly with actionable error messages
✅ **DO** have multiple implementations (flexibility is good)
✅ **DO** select implementation via explicit configuration
❌ **DON'T** downgrade silently (causes mysterious slowdowns)

### Why This Matters

Silent fallbacks are the **bane of concurrency frameworks**. They cause:
- **Mysterious slowdowns** that are hard to diagnose
- **Production surprises** when code suddenly runs 10-100x slower
- **False confidence** in development that breaks in production
- **Debugging nightmares** trying to find why "the same code" performs differently

### Real-World Example

```python
# ❌ WRONG: Silent fallback
try:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
