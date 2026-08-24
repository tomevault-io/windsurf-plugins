---
trigger: always_on
description: When the matrix runner / gate_sweep / any test harness constructs a `Gate`, it must pass `afs=(...)` (or set `WG_AFS` on the spawned subprocess) naming exactly the address families that iteration intends to exercise. The Gate validates the requested AFs against the loaded NICs' `nic.supported()` and raises `GateAfNotSupported` if any requested AF can't be served.
---

# warpgate — project instructions

## Orchestrators MUST pass explicit `afs` to `Gate`

When the matrix runner / gate_sweep / any test harness constructs a `Gate`, it must pass `afs=(...)` (or set `WG_AFS` on the spawned subprocess) naming exactly the address families that iteration intends to exercise. The Gate validates the requested AFs against the loaded NICs' `nic.supported()` and raises `GateAfNotSupported` if any requested AF can't be served.

The orchestrator side then maps that exception (signalled via the `WG_AF_NOT_SUPPORTED requested=... available=...` sentinel line from `gate_listen`) to a `SKIP_AF` outcome — distinct from `READY_FAIL` and from `echo=fail`. Aggregated reports separate **real failures** (broken cascade) from **environment-skips** (the NIC literally can't speak that AF).

Default `afs=None` keeps the historical permissive behaviour for interactive demo / single-user invocations — only the orchestrator path is constrained to explicit AFs. Without the explicit afs gate, a v6-iteration against a v4-only mobile NIC silently binds whatever the NIC offers, the cascade then fails for env-not-bug reasons, and aggregated stats misreport the env-skip as a punch regression.

Per-NIC AF capability lives in `warpgate_test_run/plugin_sweep.py:VMS[<vm>]["nic_ext_afs"]` (Windows mobile NICs are all `(4,)` — IPv4-only) and `anchor_sweep.py:VMS[<vm>]["nic_afs"]` (defaults to `(4, 6)` when absent). `matrix_full` reads these and filters its task list so the structurally-impossible combinations never even fire.

## Python compatibility

`requires-python = ">=3.5"` is intentional and must not be changed. Do not raise the minimum Python version under any circumstances.

## Dependency versions

Never add version pins to package dependencies in `setup.py`, `pyproject.toml`, or any requirements file. List packages by name only (e.g. `"ecdsa"` not `"ecdsa>=0.18"`). The only version constraint that may appear is `python_requires=">=3.5"`.

## String formatting

Never use f-string literals (`f"..."`). They require Python 3.6+ and break the 3.5 constraint. Use `.format()` or import and use `fstr(template, args_tuple)` from `aionetiface.utility.utils`:

```python
"value is {}".format(val)
fstr("value is {0}", (val,))
```

`fstr()` is a regex-based formatter and **only supports `{N}` positional placeholders**. Format-spec syntax (`{N!r}`, `{N!s}`, `{N:>5}`, `{N:.3f}`, etc.) raises `ValueError` because the regex captures the whole `1!r` and tries `int("1!r")`. If you want repr/str/formatted output, pre-format the value and pass the resulting string:

```python
# WRONG -- raises ValueError inside fstr at call time
log(fstr("name={0!r} count={1:>5}", (name, count)))

# RIGHT
log(fstr("name={0} count={1}", (repr(name), "%5d" % count)))
```

This bug bites silently because the `ValueError` from fstr in a logging call (e.g. inside an `except` handler that itself uses fstr with `!r`) cascades and can swallow the original exception — making the failure look like a hang or silent drop rather than a logging issue. Stick to plain `{N}` in every fstr template.

## Naming

Never use leading-underscore names for variables, attributes, methods, or functions (e.g. no `_foo`, `_cancel_tasks`, `_private`). Use plain names. The single exception is dunder names (`__init__`, `__all__`, etc.) which are required by Python itself.

## Print statements

Never remove or comment out `print()` calls. They are intentional debugging and observability hooks — leave them exactly as found.

## Error handling

- Use `ValueError` for invalid input at API boundaries.
- Use `AssertionError` (or bare `assert`) for internal invariants that should never be false.
- Do not use `RuntimeError` as a catch-all for invariant violations.
- Do not use `ast.literal_eval` on user-supplied input — parse it explicitly.
- Pick one error idiom per function: either return a sentinel value or raise — not both.

## Writing tests

**Never use pytest-specific code.** All tests use `unittest` with `AsyncTestCase` from `aionetiface.testing`.

### The required pattern

```python
import unittest
from aionetiface.testing import AsyncTestCase

class TestMyFeature(AsyncTestCase):
    async def asyncSetUp(self):
        # async setup — runs before each test
        self.node = await start_something()

    async def asyncTearDown(self):
        # async teardown — runs after each test
        await self.node.close()

    async def test_something(self):
        result = await self.node.do_thing()
        self.assertEqual(result, expected)

    async def test_skip_example(self):
        if condition:
            self.skipTest("reason")
        ...
```

### Rules

- Base class is always `AsyncTestCase` — never `unittest.TestCase`, `unittest.IsolatedAsyncioTestCase`, or any pytest class.
- Test methods are `async def` coroutines — the backport handles them on Python 3.5–3.7.
- Use `self.skipTest("reason")` — never `pytest.skip(...)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertsdotpm/warpgate](https://github.com/robertsdotpm/warpgate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
