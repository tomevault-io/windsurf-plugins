---
trigger: always_on
description: Testing patterns — common patterns, shared limits testing, fixing failures, organization, Ray-specific, anti-patterns.
---

# Testing Patterns and Practices

---

## Test Organization Best Practices

### 1. Group Tests by Feature

```python
class TestBasicFeatures:
    """Test basic worker functionality."""
    def test_initialization(self, worker_mode): ...
    def test_method_call(self, worker_mode): ...

class TestPoolFeatures:
    """Test worker pool features."""
    def test_load_balancing(self, pool_mode): ...
```

### 2. Use Descriptive Test Names

```python
# ✅ Good: Clear what is being tested
def test_wait_returns_done_and_not_done_sets(self, worker_mode): ...
def test_gather_preserves_input_order(self, worker_mode): ...

# ❌ Bad: Unclear
def test_wait(self, worker_mode): ...
def test_gather_works(self, worker_mode): ...
```

### 3. Always Clean Up Resources

```python
def test_feature(self, worker_mode):
    w = MyWorker.options(mode=worker_mode).init()
    result = w.method().result()
    assert result == expected
    w.stop()  # Pytest calls this even if assertion fails
```

---

## Common Testing Patterns

```python
# Pattern 1: Synchronization primitives
def test_wait_and_gather(self, worker_mode):
    w = MyWorker.options(mode=worker_mode).init()
    futures = [w.compute(i) for i in range(10)]
    done, not_done = wait(futures, timeout=5.0)
    results = gather(futures, timeout=5.0)
    w.stop()

# Pattern 2: Exception handling
def test_gather_with_exceptions(self, worker_mode):
    w = MyWorker.options(mode=worker_mode).init()
    futures = [w.compute(1), w.failing_method(), w.compute(3)]
    results = gather(futures, return_exceptions=True, timeout=5.0)
    assert isinstance(results[1], ValueError)
    w.stop()

# Pattern 3: Timeouts
def test_timeout(self, worker_mode):
    if worker_mode == "sync":
        pytest.skip("Sync mode completes immediately")
    w = MyWorker.options(mode=worker_mode).init()
    with pytest.raises(TimeoutError):
        w.slow_task(duration=5.0).result(timeout=0.1)
    w.stop()

# Pattern 4: Edge cases - empty, single, mixed types
def test_edge_cases(self, worker_mode):
    assert len(wait([])[0]) == 0  # Empty
    assert len(wait(single_future)[0]) == 1  # Single
    assert gather([future, 42, future2]) == [r1, 42, r2]  # Mixed
```

## Ray-Specific Testing

```python
# Always include runtime_env when manually initializing
import ray, morphic, concurry
ray.init(
    ignore_reinit_error=True,
    num_cpus=4,
    runtime_env={"py_modules": [concurry, morphic]}
)

# Test Ray-specific features
@pytest.mark.skipif(not _IS_RAY_INSTALLED, reason="Ray not installed")
def test_ray_actor_options(self):
    w = MyWorker.options(
        mode="ray",
        actor_options={"num_cpus": 1}
    ).init()
    w.stop()
```

---

## Fixing Failing Tests: NEVER Skip, Always Fix

### Critical Rule: Fix Implementation, Don't Skip Tests

When an existing test fails, the problem is usually the implementation, not the test. Your responsibility:
1. Identify the root cause
2. Fix the implementation to make the test pass
3. NEVER skip or comment out failing tests

### Decision Tree for Failing Tests

```
Test fails
   ├── Test wrong/outdated? → Ask user before modifying
   ├── NEW test you wrote? → Fix your test or implementation
   ├── Small fix (<50 lines, single file)? → Fix autonomously
   └── Large fix (multiple files, architectural)? → STOP and ask user
```

### When to Fix Autonomously vs. Ask User

**Small Fix (Do it)**: Off-by-one errors, missing checks, wrong operators, typos, missing imports, wrong exception types

**Large Fix (Ask user)**: New classes, API changes, 3+ files affected, architectural decisions, unclear behavior, 100+ lines of new logic

**Test Might Be Wrong (Ask user)**: Test expects value X, implementation returns Y, unclear which is correct

### Examples

✅ **Autonomous Fix**: "Test expects 3 retries but gets 2. Fixed off-by-one error in retry.py line 145."

✅ **Stop and Ask**: "Test fails due to unpicklable Lock in LimitPool. Fix requires refactoring to use multiprocessing.Manager() across 5 files. Proceed?"

✅ **Ask About Test**: "Test expects timeout=5.0 but config has 10.0. Which is correct?"

❌ **NEVER Skip**: `pytest.skip("Test fails for process mode")` - This hides bugs!

❌ **NEVER Comment Out**: `# def test_feature():` - This breaks functionality silently!

❌ **NEVER Change Test Without Understanding**: Changing assertions to match wrong behavior hides bugs!

### Acceptable Reasons to Skip Tests

1. **Feature not supported by mode**: `max_workers > 1` not supported by sync/asyncio
2. **External dependency missing**: Ray not installed, OS feature unavailable
3. **User explicitly requested**: Expensive/long-running tests temporarily disabled

### Debugging Strategy

1. Read error message: What failed? Expected vs. actual?
2. Understand test intent: What feature? What behavior?
3. Trace execution: Where does it diverge?
4. Identify root cause: Logic error? Config? Race condition?
5. Determine complexity: Small fix (do it) or large fix (ask)?
6. Implement and verify: Fix, test, check regressions

### Common Test Failure Patterns

**Timing/Race**: Test flaky → Add synchronization or increase timeout
**Mode-Specific**: Fails some modes → Fix implementation, not test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
