---
trigger: always_on
description: Function signature rules — never hide known params in **kwargs, never duplicate caller-supplied defaults.
---

# Function Signature Rules

## Never Hide Known Parameters in `**kwargs`

**If you know a parameter's name at the time you write the function, it MUST be an explicit parameter with a type annotation.** Using `kwargs.get("some_name")` to extract a value that the function *always expects* is as bad as `.get(key, None)` — it silently returns `None` instead of raising, hides the interface from callers, and completely defeats type checking.

This is not a style preference. It is a correctness issue:

1. **`@validate` and `@field_validator` cannot see kwargs.** Morphic's `@validate` decorator and Pydantic validators use the function signature to enforce types at runtime. A parameter extracted via `kwargs.get()` is invisible to them — it arrives as an untyped `Any` inside a `dict`, bypassing every safety net.
2. **Static type checkers are blind.** mypy, pyright, and IDE autocomplete cannot infer the type of `kwargs.get("timeout")`. The caller gets no autocomplete, no type errors, no hover documentation.
3. **Silent `None` on missing keys.** `kwargs.get("timeout")` returns `None` when the caller forgets to pass `timeout`. The function proceeds with `None`, producing a confusing `TypeError` or wrong behavior ten lines later instead of a clear `TypeError: missing required argument` at the call boundary.
4. **The function signature lies.** When you read `def submit(self, **kwargs)`, you have no idea what the function actually needs. You must read the implementation to discover that it does `kwargs.get("timeout")`, `kwargs.get("retry_config")`, etc. This is a hidden interface.

❌ Bad (known parameters hidden in `**kwargs`):
```python
def execute(self, task: Task, **kwargs) -> Future:
    timeout = kwargs.get("timeout")           # ❌ Hidden param, silent None
    retry_config = kwargs.get("retry_config") # ❌ Hidden param, silent None
    priority = kwargs.get("priority", 0)      # ❌ Hidden default

    if timeout is not None:          # ❌ Now you need a None guard
        self._set_timeout(timeout)
```

✅ Good (all known parameters are explicit):
```python
def execute(
    self,
    task: Task,
    *,
    timeout: Optional[float] = None,
    retry_config: Optional[RetryConfig] = None,
    priority: int = 0,
    **kwargs,
) -> Future:
    if timeout is not None:
        self._set_timeout(timeout)
```

**The rule is simple:** if you are writing `kwargs.get("some_name")` and `some_name` is a string literal you typed yourself, that parameter MUST be promoted to an explicit function parameter. `**kwargs` is for truly unknown, pass-through parameters (e.g., forwarding to a parent class or a third-party library). It is never a substitute for declaring your interface.

**Corollary — `**kwargs` in an abstract method signature:** When a base class defines `def process(self, ..., **kwargs)` and every subclass does `kwargs.get("timeout")`, the base class signature is wrong. Add `timeout: Optional[float] = None` to the base class and all subclasses. The `**kwargs` should only carry genuinely subclass-specific extensions that differ across implementations and cannot be enumerated on the base.

## Never Duplicate Caller-Supplied Values as Parameter Defaults (CRITICAL)

**When a parameter is always supplied by the caller, it MUST NOT have a default value on the receiving function.** A default on a parameter that the caller always passes is dead code that silently masks a broken call chain. If the caller stops passing the value (due to a refactoring bug, a missing context key, or a dict-splatting error), the function falls back to the hard-coded default instead of crashing. The developer never learns that the value they configured is not reaching the function.

**This is the function-signature equivalent of `.get(key, default)`.** The `.get()` anti-pattern invents a fallback value when a dict key is missing. A redundant parameter default invents a fallback value when a caller forgets to pass an argument. Both hide the same bug: a value that should be present is absent, and the program continues with wrong data instead of crashing.

**Why LLMs produce this pattern:** LLM coding assistants generate parameter defaults reflexively because they cannot see the call chain. When an LLM writes `def execute(self, *, timeout: float = 30.0, ...)`, it does not know that `timeout` is always passed from `global_config` at the call site. The LLM adds a default "just in case," which is exactly the defensive-coding instinct that produces `.get(key, default)` in dicts. The result is the same: a silent fallback that masks bugs.

**The decision rule:** ask **"Does every caller always pass this argument?"**
- **Yes** (the value comes from a config object, a builder, or an explicit call site) → **no default**. Remove it. If the caller ever fails to pass it, the function must crash with `TypeError: missing required keyword argument`.
- **No** (some callers genuinely omit the argument, and `None`/a sentinel is a valid state) → a default is appropriate.

**Three forms of this anti-pattern:**

### Form 1: Parameter default duplicates a config-supplied value (most dangerous)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
