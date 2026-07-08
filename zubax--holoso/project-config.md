---
trigger: always_on
description: Holoso converts a small subset of Python functions and expressions into synthesizable and verifiable Verilog.
---

# Holoso -- simple high-level synthesis of Python into Verilog for numerical code

Holoso converts a small subset of Python functions and expressions into synthesizable and verifiable Verilog.
Read the `README.md` and `DESIGN.md`.

Whenever introducing nontrivial changes, update `DESIGN.md` as well to keep it fully up-to-date and non-conflicting
with the implementation. However, do not attempt to capture minor implementation minutiae there, keep it high-level.

Do not commit anything unless asked explicitly to do so.

When (sub)agents fail or get stuck, e.g. due to a connection error or a transient environment error,
retry them until success.

## Conventions

Given a trade-off between performance and simplicity, always choose simplicity.
Clear designs are easier to verify, maintain, and refactor, and they are more likely to be correct.

Do not write any comments or docs unless they add something that is impossible to infer from the source code,
such as design rationale, high-level context, non-obvious implications, etc.
Comments that describe what the code does or restate what is inferrable from the type system are strictly prohibited.

### Reset strategy

Use synchronous active-high reset for stream control only: validity flags, state-machine state, and other control
registers that define whether an output transaction is meaningful.
Avoid resetting pure datapath registers whose contents are ignored while their associated valid flag is deasserted.
This keeps high-fanout reset nets out of wide payload cones, reduces control-set pressure,
and gives synthesis/place-and-route more freedom to retime and optimize pipeline registers.

Do not write the datapath assignment only in the reset-else branch, as it still makes data depend on reset because
the register is held during reset. A better strategy is to make datapath manipulation reset-unconditional
and only keep the control signals under reset/else.

References:

- AMD UG949, "When and Where to Use a Reset":
  <https://docs.amd.com/r/en-US/ug949-vivado-design-methodology/When-and-Where-to-Use-a-Reset>
- Intel Hyperflex Architecture High-Performance Design Handbook, "Synchronous Resets Summary":
  <https://docs.altera.com/r/docs/683353/25.1.1/hyperflex-architecture-high-performance-design-handbook/synchronous-resets-summary?contentId=vgtR8yUs_Z5DH0ApHJFiTQ>
- Intel Hyperflex Architecture High-Performance Design Handbook, "Reset Strategies":
  <https://docs.altera.com/r/docs/683353/25.1.1/hyperflex-architecture-high-performance-design-handbook/reset-strategies?contentId=gzd92HdsL40qZGHurB0ezg>

### Python

Follow PEP8 with one exception: the maximum line length is 120 columns. This is already configured in Black.

Use strongly typed primitives. Instead of int constants, prefer enums; instead of dicts, prefer dataclasses;
instead of existence/vaidity flags, prefer optional type or unions, etc.

If a design calls for a leaky abstraction, discard it and redesign from scratch, even if it involves breaking changes.
This means that even implementing a seemingly minor change may warrant a deep refactor if the change doesn't fit
cleanly into the existing architecture.
An acceptable design will not involve special-casing. Do not bypass existing abstractions to get the job done.
Work will not be accepted unless architected cleanly.

When editing code, do not ever leave any compatibility shims behind. Always do a clean break with bridges burned.
API compatibility is not a concern.

The Python version to target is 3.14 and newer. No need to ensure compatibility with older versions.

Do not use `from __future__ import annotations`.
Prefer `list` over `tuple[X, ...]` for homogeneous sequences (unless immutability is required).
Do not use `Protocol`, prefer `ABC`/`@abstractmethod` instead for interfaces and abstract base classes.
Prefer explicit `from X import Y as Y` instead of using `__all__` in `__init__.py` files.

Public APIs can only include items that are required to use the API and nothing else.
All non-public items are hidden in underscore-prefixed submodules.
Minimize the public API surface.

Importing anything from a package or subpackage is only allowed as long as it doesn't involve referencing
underscore-prefixed names. Exceptions apply for importing from parent modules with the dot notation, and for unit tests.
Accessing underscore-prefixed names from outside a class (or its descendants) is not allowed;
all externally accessble entities must be non-underscore-prefixed.

For invariant checking use plain `assert` statements copiously; usually any nontrivial function should contain at
least a few. Their disappearance under `-O` is intentional.
The assertion description string is only useful in nonobvious cases;
do not spell out what is clear from the asserted expression. 

Avoid files longer than about ~2000 lines (this is a soft limit).
If a file grows beyond that, consider refactoring into smaller modules.

If a docstring comment doesn't fit on one line, add an initial line break like this:

```python
"""
This is a very long
comment string.
"""
```

Instead of this:

```python
"""This is a very long
comment string.
"""
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zubax/holoso](https://github.com/Zubax/holoso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
