---
trigger: always_on
description: Morphic Typed & Registry patterns — anti-patterns, aliases, ClassVar, classproperty, PrivateAttr.
---

# Morphic Typed & Registry Patterns

Concurry's data model is built on `morphic.Typed` (immutable Pydantic BaseModel) and `morphic.Registry` (inheritance-based class registration with string-based factory lookup). Understanding these two base classes is essential to writing idiomatic Concurry code.

## Quick Reference: What Typed Gives You for Free

| Feature | How | Manual Code It Replaces |
|---|---|---|
| Immutability | `frozen=True` by default | No need for `__slots__`, `@dataclass(frozen=True)` |
| Validation on construction | Pydantic field types | Manual `if not isinstance(...)` checks |
| Type coercion | Automatic (`"25"` → `int(25)`) | Manual `int(x)` calls |
| Serialization | `.model_dump()` → dict | Hand-written `.to_dict()` methods |
| Factory method | `MyClass.of(...)` or `Base.of("subclass", ...)` | Manual `__init__` wiring |
| Lifecycle hooks | `pre_initialize`, `pre_validate`, `post_initialize`, `post_validate` | `__init__` overrides with `object.__setattr__` |
| Registry lookup | `Base.of("thread", ...)` | Manual if/elif chains or dict dispatches |
| Nested model coercion | `dict` → `Typed` subclass automatically | Manual `MyModel(**d)` construction |

## Anti-Pattern 1: Hand-Written `to_dict()` Methods

`Typed` inherits Pydantic's `.model_dump()` which serializes all fields to a dict recursively. Never write a custom `to_dict()` on a `Typed` subclass.

❌ Bad (hand-written serialization on a Typed subclass):
```python
class RetryConfig(Typed):
    max_retries: int
    wait_seconds: float
    algorithm: str

    def to_dict(self) -> Dict[str, Any]:
        return {
            "max_retries": self.max_retries,
            "wait_seconds": self.wait_seconds,
            "algorithm": self.algorithm,
        }
```

✅ Good (use the built-in):
```python
class RetryConfig(Typed):
    max_retries: int
    wait_seconds: float
    algorithm: str

# Call sites:
config_dict = retry_config.model_dump()
```

**Exception:** If you need to *exclude* certain fields or rename keys for a specific serialization context, use `model_dump(include=..., exclude=...)` rather than writing a custom method.

## Anti-Pattern 2: Using `object.__setattr__` to Mutate Typed Fields

`Typed` is frozen (immutable). Calling `object.__setattr__(self, "field", value)` bypasses Pydantic validation and breaks the immutability contract. If you need to normalize a field during construction, use `model_post_init` (Pydantic's built-in hook) or Typed's `pre_initialize` lifecycle hook.

❌ Bad (bypassing frozen with `object.__setattr__` for field normalization):
```python
class LimitPool(Typed):
    load_balancing: Optional[str] = None

    def post_initialize(self) -> None:
        if self.load_balancing is None:
            local_config = global_config.clone()
            object.__setattr__(self, "load_balancing", local_config.defaults.limit_pool_load_balancing)
```

✅ Good (normalize in `pre_initialize` before Pydantic validates):
```python
class LimitPool(Typed):
    load_balancing: str  # Always resolved after construction

    @classmethod
    def pre_initialize(cls, data: dict) -> None:
        if data.get("load_balancing") is None:
            local_config = global_config.clone()
            data["load_balancing"] = local_config.defaults.limit_pool_load_balancing
```

**When `object.__setattr__` IS acceptable:**
- On `MutableTyped` subclasses (which are not frozen) — e.g., `GlobalDefaults`, `PromptMOOConfig`
- Setting `PrivateAttr` values in `post_initialize` (Pydantic explicitly supports this)
- On non-Typed objects where you are stamping runtime metadata

## Anti-Pattern 3: Mutable State as Typed Fields with `= None`

`Typed` fields are frozen after construction. Storing mutable runtime state (queues, locks, processes, balancer instances) as regular fields and then assigning them in `post_initialize` via `object.__setattr__` works but is semantically wrong — it says "this is an immutable config field" when it is actually mutable runtime state. Use Pydantic's `PrivateAttr` for mutable internal state on frozen models.

Concurry already does this correctly in most places — follow the existing pattern:

✅ Good (from `ProcessWorkerProxy`):
```python
from pydantic import PrivateAttr

class ProcessWorkerProxy(WorkerProxy):
    _command_queue: Any = PrivateAttr()
    _result_queue: Any = PrivateAttr()
    _futures: dict = PrivateAttr()
    _futures_lock: Any = PrivateAttr()
    _process: Any = PrivateAttr()
    _result_thread: Any = PrivateAttr()
```

✅ Good (from `LimitPool`):
```python
class LimitPool(Typed):
    _balancer: Any = PrivateAttr()
```

**Rule of thumb:** If a field is set once at construction and never changes, it is a Typed field. If it is created/mutated during the object's lifetime (locks, queues, runtime handles, balancers), it must be a `PrivateAttr`.

## Anti-Pattern 4: Manual Registry Dispatch Instead of `Registry.of()`

The `Registry` pattern provides `Base.of("key", **kwargs)` which resolves the correct subclass by name/alias. Never write manual if/elif dispatch chains or dict-based lookups to select subclasses.

❌ Bad (manual dispatch):
```python
if mode == "thread":
    proxy = ThreadWorkerProxy(...)
elif mode == "process":
    proxy = ProcessWorkerProxy(...)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
