---
trigger: always_on
description: These rules apply to ALL newly written or modified code. Follow them even when
---

# AlpaGym — AI Agent Context

## Coding Principles

These rules apply to ALL newly written or modified code. Follow them even when
surrounding code uses different patterns.

### Rationale

We aim for a *lightweight* codebase that is easy to read, understand and modify.
Our target audience are researchers with their unique ideas and setup who will
likely not use the code exactly as is, but will need to understand and modify it.

### Guidelines

- **Readability over flexibility.** Introduce abstractions only when they reduce
  net complexity (the abstraction is simpler to understand than the in-lined
  code). Don't prematurely introduce them (YAGNI).
- **Minimal set of features.** Avoid bloat and feature creep, instead focusing
  on the core functionality. We also don't aim to cover all eventualities and
  edge cases. Solve the present problem, not a hypothetical future one.
- **One way of doings things.** Be opinionated and aggressively remove legacy
  code paths. Avoid special cases. Even for different configurations (e.g.
  different drivers or deployments), try to unify APIs as much as possible.
- **Locality of behavior.** Avoid indirections, one-line wrapper functions,
  gratuitous delegation, and unnecessary class hierarchies. Every "hop" must
  earn its keep by improving readability or enabling genuine reuse. For example,
  when extracting code into a function, ask whether it is easier to understand
  the abstraction (e.g. function name) or the inlined code. If the latter, keep
  it inlined. In this case, it is likely also not critical to be tested in
  isolation.
- **Configuration in one place.** Single source of truth for all config
  (proposed: YAML + Hydra). No env-variable overrides, no parallel CLI args, no
  `constants.py` defaults that shadow the real config.
- **Method docstrings.** All methods should have docstrings. Public methods
  should have detailed docstrings that describe their behavior, arguments, and
  return values. Private methods should still be documented, but the level of
  detail should match the method's complexity.
- **Google Python Style Guide** as general baseline.

### Rules

- Avoid `getattr` + raise-on-empty blocks for defensively accessing attributes.
- In docstrings, prefer simple, clear explanations over jargon-rich descriptions.
- Explicit local call signatures: Do not introduce dataclasses that only
  bundle arguments for a local method or helper. Prefer explicit keyword
  parameters so the callee signature shows what it needs. Use dataclasses for
  real domain objects, protocol payloads, configuration schemas, queue messages,
  and persisted artifacts, not for synthetic `FooRequest` or `FooArgs` argument
  bags.
- Explicit local types where inference is weak: Annotate local variables
  when assigning the return value of an untyped, loosely typed, generated,
  factory, or boundary-crossing call, especially when the concrete type is
  important for readability or static checking. Apply this only when the
  annotation is lightweight and names a useful concrete type. Do not add
  annotations that require extra `cast(...)` calls, spell only `object` or
  similarly broad types, duplicate types already clear from a constructor or
  strongly typed helper, or otherwise make local code harder to read.

### Examples

#### Error Handling

- Fail fast, never silently fall back
- Raise an error on unexpected input. Do not default-away problems. Rely on
  default exceptions (e.g. `KeyError`) rather than catching and re-raising with
  a custom message.

```python
# DISCOURAGED -- silent fallback hides bugs
def get_backend(name: str) -> Backend:
    return BACKENDS.get(name, DefaultBackend())

# DISCOURAGED -- fails fast, but with an unnecessary wrapper that adds no value
def get_backend(name: str) -> Backend:
    if name not in BACKENDS:
        raise ValueError(f"Unknown backend: {name!r}")
    return BACKENDS[name]

# PREFERRED -- fail fast and minimize abstraction layers
backend = BACKENDS[name]
```

#### Minimize abstraction layers

Add a new class/wrapper/layer only when it removes duplication across 3+
call sites. One caller = inline the logic.

```python
# DISCOURAGED -- unnecessary wrapper for a single use
class QueryExecutor:
    def __init__(self, db):
        self.db = db
    def run(self, sql):
        return self.db.execute(sql)

# PREFERRED -- call directly
result = db.execute(sql)
```

#### Minimize the use of constants

Inline literals by default. Prefer reading the real value at the call site over
chasing named constants. Do not introduce constants, constant classes, or
dataclass “namespaces” for file-local strings, numbers, prefixes, or small path
fragments. This is stricter than the general repo baseline: reuse alone does
not justify extracting a constant in this project.

Introduce a named constant only in rare cases where multiple components must
agree on the exact same value and inlining it in each place would risk hard to
diagnose/detect drift. Typical examples are env var names or schema keys shared
by producer and consumer, and file or directory names used across multiple
components. A value being part of an external interface is not, by itself,
enough to justify a constant.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVlabs/alpagym](https://github.com/NVlabs/alpagym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
