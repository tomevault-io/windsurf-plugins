---
trigger: always_on
description: Guidance for AI agents working in this repository. `GEMINI.md` and `AGENTS.md`
---

# Agent guide: switchlang (python-switch)

Guidance for AI agents working in this repository. `GEMINI.md` and `AGENTS.md`
are symlinks to this file — edit this one.

## What this is

`switchlang` adds an explicit **switch statement to Python without changing the
language**. It is implemented as a context manager: you open a `with switch(value)`
block, register cases as method calls, and read the matched case's return value
from `s.result`. The whole library is ~200 lines with **zero runtime dependencies**.

- **Repo name** is `python-switch`; the **PyPI / import name** is `switchlang`.
  Don't confuse the two — `import switchlang`, `pip install switchlang`.
- The package ships type hints and a `py.typed` marker (PEP 561). It is
  `Typing :: Typed`.
- **The default git branch is `master`, not `main`.** Several tools (and
  `great-docs.yml`) hard-code this.

## Repository layout

| Path | What it is |
|------|------------|
| [switchlang/__init__.py](switchlang/__init__.py) | Public package: re-exports `switch` and `closed_range`, sets `__version__`/`__author__`/`__all__`. |
| [switchlang/__switchlang_impl.py](switchlang/__switchlang_impl.py) | **The entire implementation** — `switch` class and `closed_range()`. Make code changes here. |
| [switchlang/py.typed](switchlang/py.typed) | PEP 561 marker so type checkers read our hints. |
| [tests/test_core.py](tests/test_core.py) | The whole test suite (29 `unittest`-style tests). |
| [pyproject.toml](pyproject.toml) | Packaging (hatchling), metadata, version, `[dev]` extra. |
| [ruff.toml](ruff.toml) | Lint + format config. |
| [great-docs.yml](great-docs.yml) | Docs site config (Great Docs / Quarto). |
| [scripts/build_docs.py](scripts/build_docs.py) | Builds docs and mirrors them into `docs/`. |
| [scripts/serve_docs.py](scripts/serve_docs.py) | Local preview that mimics the nginx subpath. |
| [docs/](docs/) | **Generated** static site (committed, served at `mkennedy.codes/docs/python-switch`). Do not hand-edit. |
| [README.md](README.md) | The narrative/marketing docs and rationale. Keep in sync with behavior. |

Generated / ignored artifacts you should not edit or commit by hand: `docs/`
(regenerate it), `great-docs/` (ephemeral build dir, gitignored), `dist/`,
`*.egg-info/`, `venv/`.

## The public API and its semantics

Only two names are public (`switchlang.__all__`): **`switch`** and **`closed_range`**.

```python
from switchlang import switch, closed_range

with switch(value) as s:
    s.case('a', process_a)                       # key == value -> run process_a
    s.case(['v', 'b'], view_bookings)            # list key: each item is a case
    s.case(range(1, 6), handler)                 # range key: each item is a case
    s.case(closed_range(1, 5), handler)          # inclusive range: 1,2,3,4,5
    s.case(2, do_two, fallthrough=True)          # opt into running the next case too
    s.default(unknown_command)                   # runs if nothing else matched
print(s.result)                                  # return value of the executed case
```

Behaviors that are easy to get wrong — preserve all of these (they are pinned by tests):

- **Cases run on block exit, not at registration.** `case()`/`default()` only
  *register*; the matched function(s) execute in `__exit__`. So `s.result` is only
  valid *after* the `with` block. Reading it inside the block raises.
- **Matching is equality-based** (`key == value`). Keys are also stored in a `set`,
  so case keys must be **hashable**. Any hashable value works as a key, including
  `None` and arbitrary objects.
- **`default()` is just a case** keyed on a private sentinel, and **ordering is not
  enforced**: a default registered *before* a matching case will also run. **Always
  register `default()` last.**
- **`result` uses identity, not equality, against its "no result" sentinel.** A
  computed result with a permissive `__eq__` (e.g. a NumPy array) must not be
  mistaken for "nothing computed." `None` is a valid computed result and is
  distinct from "not computed."
- **`case()` returns `bool`** — `True` if the case (or any item of a list/range
  key) matched.
- **Fall-through is opt-in** per case via `fallthrough=True`; the next registered
  case then runs whether or not its key matches, and so on until a case without
  fall-through. When falling through, `result` is the **last** function executed.
  The `fallthrough=None` value is **reserved for internal recursion** (list/range
  expansion) and must not be used by callers.
- **`closed_range(start, stop, step=1)`** is inclusive on **both** ends and never
  overshoots `stop`: `closed_range(1, 5)` -> `1,2,3,4,5`; `closed_range(1, 6, 2)`
  -> `1,3,5`; `closed_range(1, 7, 2)` -> `1,3,5,7`. Note adjacent closed ranges
  **overlap** (`closed_range(1,5)` and `closed_range(5,9)` both contain 5) and will
  raise a duplicate-case error.

Validation (all raise on registration/exit):

- Duplicate case key -> `ValueError`.
- `func` that is `None` or not callable -> `ValueError`.
- Empty list/range key (`[]`) -> `ValueError` (it could never match).
- `closed_range` with `start >= stop` or `step < 1` -> `ValueError`.
- No case matched **and** no `default()` registered -> `Exception` on block exit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikeckennedy/python-switch](https://github.com/mikeckennedy/python-switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
