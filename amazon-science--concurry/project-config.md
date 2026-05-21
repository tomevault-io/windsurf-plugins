---
trigger: always_on
description: Loud failures over silent defaults — the .get(key, default) anti-pattern and all its variants.
---

# Loud Failures Over Silent Defaults (CRITICAL — Read This First)

**A program that crashes on misconfiguration is infinitely better than a program that silently produces wrong output.** This is the single most important principle in this codebase. Every other rule — fail-fast, strict typing, `@validate`, exhaustive dispatch, the `Any` ban — is a corollary of this principle.

**Why this is non-negotiable:** A silent wrong output destroys trust. If a worker pool silently falls back to a slower implementation because a configuration is wrong, nobody knows until mysterious slowdowns appear in production — if they ever notice. A crash at initialization with a clear error message costs thirty seconds to fix. A silent degradation can cost days of debugging.

**The test is simple:** when you write code that handles a missing or unexpected value, ask: **"If this value is wrong, would I rather crash immediately or produce incorrect/degraded behavior that looks correct?"** The answer is always crash. No one pushes code to production without testing. A crash during testing is a gift — it tells you exactly what to fix. A silent default that degrades behavior is a bomb with a delayed fuse.

LLM coding assistants are the primary source of silent-default code. They are trained on web application code where `.get(key, default)` is a reasonable defensive pattern — web apps must not crash on a single bad request. In a library/infrastructure codebase, the opposite is true: **the system MUST crash on bad configuration** so the developer fixes it before deployment.

## The `.get(key, default)` Anti-Pattern

**Never use `dict.get(key, hardcoded_default)` when the key is expected to exist.** This is the most common form of silent default. It tells the reader "this key might be absent, and that's fine." If the key *should* be present, its absence is a bug — and `.get()` with a fallback hides that bug.

**The decision rule:** ask "if this key is missing, is that a valid state or a bug?"
- **Valid state** (sparse data where absence is meaningful) → `.get(key)` returning `None`, with explicit handling of the `None` case
- **Bug** (the key should always be present given the program's logic) → use bracket access `dict[key]`, or check-and-raise with feedback

❌ Bad (silent default hides missing key):
```python
timeout = config.get("timeout", 30.0)
```

✅ Good (check, raise with feedback data, then access):
```python
if "timeout" not in config:
    raise ValueError(
        f"Expected 'timeout' in config, "
        f"but only found keys: {list(config.keys())}"
    )
timeout = config["timeout"]
```

✅ Good (bracket access — crashes immediately with a `KeyError` that names the missing key):
```python
timeout = config["timeout"]
```

✅ Best (use a Typed class so the key cannot be missing — Pydantic enforces it at construction):
```python
class WorkerConfig(Typed):
    timeout: float

config = WorkerConfig(**raw_config)  # Pydantic raises ValidationError if timeout missing
timeout = config.timeout  # Guaranteed to exist
```

## Feedback-Driven Exceptions

When raising an exception, **include the actual data** that caused the failure. The error message is feedback to the developer — it should contain enough information to diagnose and fix the problem without a debugger.

The pattern: **state what was expected, state what was received, and show the available options.**

❌ Bad (error message gives no feedback):
```python
raise ValueError("Invalid worker mode")
```

✅ Good (expected + received + available):
```python
raise ValueError(
    f"Unknown worker mode {mode!r}. "
    f"Must be one of: {list(ExecutionMode)}."
)
```

✅ Good (Pydantic/Typed already does this — leverage it):
```python
# Pydantic's ValidationError automatically includes:
# - Which field failed
# - What value was provided
# - What type/constraint was expected
# This is WHY we use @validate and Typed — they produce feedback-driven errors for free.
```

**This pattern connects to every other rule in this document:**
- **Fail Fast** (below) is the timing corollary: crash at the earliest possible point.
- **`@validate`** is the automated version: Pydantic produces feedback-driven `ValidationError` messages.
- **Exhaustive dispatch with raising `else`** catches unhandled values at dispatch time.
- **Never Hide Known Parameters in `**kwargs`** prevents silent `None` from `.get()`.
- **Never Duplicate Caller-Supplied Values as Parameter Defaults** prevents dead-code defaults from masking broken call chains.
- **Morphic `Typed` fields** replace `dict` access with validated attribute access — `.get()` is impossible on a Typed instance.
- **No Silent Fallbacks** (in concurry-architecture.mdc) is the infrastructure-level application of this same principle.

## The Silent-Default Family (All Variants)

`.get(key, default)` is the most common silent-default pattern, but it belongs to a family of related patterns that all share the same flaw: **they invent data when data is missing, instead of surfacing the absence as an error.** LLMs produce all of these reflexively. When you see any of them, apply the same decision test: "is the absence a valid state, or a bug?"

| Silent-Default Pattern | What it hides | Loud alternative |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
