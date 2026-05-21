---
trigger: always_on
description: Type hints, the Any ban, Optional/Union scrutiny, and variable annotation rules.
---

# Type Hints and Typing Rules

## Type Hints
- **Always include type hints** for all function parameters and return values
- **Always use capitalized `typing` imports**: `List`, `Dict`, `Set`, `Tuple`, `Optional`, `Union`, `FrozenSet`
- Do NOT use lowercase built-in generics (`list[str]`, `dict[str, int]`) or pipe unions (`str | None`) even though Python 3.10+ supports them
- **Rationale:** Capitalized typing imports are visually distinct and grep-able. `List` is unambiguously a type annotation; `list` could be a variable name, a builtin call, or a type hint. This makes bulk find-and-replace across files reliable, and makes it immediately clear when reading code that a line involves type annotations.
- Use `-> None` for functions that return no value (side-effect functions, lifecycle hooks like `post_initialize`, `on_start`, `on_stop`)
- Use `-> NoReturn` ONLY for functions that genuinely never return (unconditional `raise`, `sys.exit()`, infinite loops)
- Include type hints for class attributes and instance variables
- **Prefer `morphic.Typed` over `TypedDict`** for structured data. `TypedDict` is a fallback for cases where `Typed` cannot be used (e.g., dicts that must remain plain `dict` for JSON serialization to a third-party API, or when the consuming code expects a raw dict, not a model). When you control both producer and consumer, use `Typed`.
- **`Protocol` is for function parameters and call-site typing, NOT for Pydantic/Typed field types.** Pydantic v2 validates Protocol fields with `isinstance()` at construction time, which (a) rejects `None` for non-Optional fields even in tests that don't exercise the field, and (b) fails on proxy objects that use `__getattr__`-based dynamic dispatch (e.g., Concurry `Worker` proxies) because Python's `isinstance` Protocol check inspects the type's `__dict__` and MRO, not dynamic attributes. Declare such fields as `Any` and document the duck-typed contract in a comment referencing the Protocol class. The Protocol provides type safety at *call sites* (IDE autocomplete, static analysis); Pydantic field validation is the wrong enforcement point.

### Strict `Any` Ban (CRITICAL)

**`Any` is as good as no typing.** Using `Any` removes the entire reason to use Morphic Typed, Pydantic, and type hints. It tells neither the reader nor the type checker what the value actually is. LLM coding assistants produce `Any` reflexively because it silences type errors without solving them.

**`Any` is ONLY acceptable in these specific situations:**

| Situation | Why `Any` is acceptable | Example |
|---|---|---|
| **Pydantic/Typed fields for duck-typed workers** | Pydantic's `isinstance()` validation rejects Concurry `WorkerProxy` proxy objects; the Protocol cannot be used as a field type. | `worker: Any  # WorkerProtocol; see types.py` |
| **`_serialize_value`-style functions** | Serialization functions that genuinely accept any Python object. | `def serialize(value: Any) -> Any:` |
| **`Dict[str, Any]` for genuinely heterogeneous dicts** | When the dict's values are mixed types by design (JSON payloads, config dicts, serialized output). | `metadata: Dict[str, Any]` |
| **`**kwargs: Any`** | Catch-all for pass-through kwargs to parent classes or third-party libraries. | `**kwargs: Any` |
| **`PrivateAttr` for opaque runtime handles** | Locks, queues, processes, event loops — objects from external libraries whose type is complex or private. | `_lock: Any = PrivateAttr()` |

**`Any` is NEVER acceptable in these situations:**

| Situation | What to use instead | Example fix |
|---|---|---|
| **Function parameters where you know the type** | Use the actual type. | `task: Task` not `task: Any` |
| **Return types where you know the type** | Use the actual type. | `-> List[Future]` not `-> List[Any]` |
| **List/Dict generic parameters** | Spell out the element type. | `List[WorkerProxy]` not `List[Any]` |
| **Forward references within the same package** | Use string annotations or fix import ordering. See the Forward References rule below. | `result: "WorkerResult"` not `result: Any` |

**The test is simple:** when you write `Any`, ask: **"Do I know what type this actually is at runtime?"** If yes, write that type. If you are writing `Any` to avoid an import or to silence a type error, fix the import or the type error instead.

❌ Bad (known types hidden behind `Any`):
```python
def submit_batch(
    self,
    tasks: List[Any],              # ❌ These are Task objects
    callback: Optional[Any] = None,  # ❌ This is Callable[[Result], None]
) -> List[Any]:                    # ❌ These are Future objects
```

✅ Good (actual types everywhere they are known):
```python
def submit_batch(
    self,
    tasks: List[Task],
    callback: Optional[Callable[[Result], None]] = None,
) -> List[Future]:
```

### No `Any` for Forward References (CRITICAL)

**Never use `Any` as a workaround for forward references.** This defeats the entire purpose of Morphic Typed — if the field is typed `Any`, Pydantic cannot validate it, the IDE cannot autocomplete it, and the reader does not know what it holds.

**Forward references within the same file:** Use a string annotation (`"ClassName"`) or add `from __future__ import annotations` at the top of the file.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
