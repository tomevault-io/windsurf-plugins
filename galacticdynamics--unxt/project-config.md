---
trigger: always_on
description: `unxt` is unitful quantities and calculations in JAX, built on [Equinox](https://github.com/patrick-kidger/equinox), [Quax](https://github.com/nstarman/quax), [quaxed](https://github.com/GalacticDynamics/quaxed), and [quax-blocks](https://github.com/GalacticDynamics/quax-blocks), with [plum](https://github.com/beartype/plum) multiple dispatch throughout. It's the foundation quantity library other GalacticDynamics packages (coordinax, galax) build on.
---

# unxt — Agent Instructions

`unxt` is unitful quantities and calculations in JAX, built on [Equinox](https://github.com/patrick-kidger/equinox), [Quax](https://github.com/nstarman/quax), [quaxed](https://github.com/GalacticDynamics/quaxed), and [quax-blocks](https://github.com/GalacticDynamics/quax-blocks), with [plum](https://github.com/beartype/plum) multiple dispatch throughout. It's the foundation quantity library other GalacticDynamics packages (coordinax, galax) build on.

For _using_ `unxt` correctly — `Quantity` vs `ParametricQuantity`, dispatch gotchas, the `_mk` hazard — read [skills/unxt/SKILL.md](skills/unxt/SKILL.md). This file is for working _inside_ this repo.

## Essential commands

```bash
uv sync --group dev --extra all      # install, all extras + dev tooling
uv run nox -s all                    # the full gate: lint -> test -> docs
uv run nox -s lint                   # pre-commit (incl. pyright/ty/mypy) + pylint
uv run nox -s test                   # pytest, every workspace package
uv run nox -s pytest -- unxt         # pytest for one package only (see PackageEnum)
uv run nox -s docs -- --serve        # build + preview the Sphinx site
uv run nox -s docs -- -b linkcheck   # check doc links
uv run nox -s pytest_benchmark       # CodSpeed benchmarks (also gated by the `run-benchmarks` PR label)
```

Always go through `uv run`/`nox` — never bare `python`/`pytest`/`ruff`. Sync first if `uv.lock` moved.

## Workspace layout

Root `unxt` (`src/unxt/`) plus a `uv` workspace at `packages/*`. Two package families, same functionality, **different names — don't confuse them**:

| Family | Packages | Status |
| --- | --- | --- |
| **canonical** (dotted, `unxts.*`) | `unxts.api`, `unxts.hypothesis`, `unxts.interop.gala`, `unxts.interop.matplotlib`, `unxts.interop.xarray`, `unxts.linalg`, `unxts.parametric` | current, since v2.0.0 |
| **legacy shims** (hyphenated, `unxt-*`) | `unxt-api`, `unxt-hypothesis` | back-compat re-exports of `unxts.api`/`unxts.hypothesis`; new code should depend on the canonical package |

New functionality goes in a canonical `unxts.*` package, never in a shim. Release tags are hyphenated even for dotted packages (`unxts.api` → tag `unxts-api-vX.Y.Z`) — see [RELEASING.md](RELEASING.md).

| Package | Provides |
| --- | --- |
| `unxt` (root) | `Quantity`/`Q`, `Angle`, `StaticQuantity`, units, dims, unit systems, plum dispatch API |
| `unxts.api` | abstract dispatch interfaces (`uconvert`, `ustrip`, `unit`, `dimension`, ...), minimal deps |
| `unxts.hypothesis` | Hypothesis strategies for property-based testing of quantities |
| `unxts.parametric` | `ParametricQuantity`/`PQ` — dimension baked into the type, runtime-checked |
| `unxts.linalg` | `QuantityMatrix`/`QM`, `UnitsMatrix` — heterogeneous-unit linear algebra |
| `unxts.interop.gala` | `gala.units.UnitSystem` ↔ unxt `UnitSystem`, via `plum.conversion_method` |
| `unxts.interop.matplotlib` | `matplotlib.units.ConversionInterface` for plotting quantities |
| `unxts.interop.xarray` | xarray accessors/conversion for quantities |

## Architecture

`AbstractQuantity` (a `quax.ArrayValue`, so it's a JAX PyTree via Equinox) is the base of the whole hierarchy:

- **`Quantity`/`Q`** (default, root `unxt`) — non-parametric: one class, one pytree node type, for every physical dimension. No dimension checking at construction.
- **`ParametricQuantity`/`PQ`** (`unxts.parametric`, opt-in) — dimension encoded in the type (`PQ["length"]`), a distinct pytree type per dimension, runtime-checked at construction.
- **`BareQuantity`** — **deprecated** alias of `Quantity`; don't reintroduce it in new code (see [docs/reference/glossary.md](docs/reference/glossary.md), [docs/how-to/migrate-to-v2.md](docs/how-to/migrate-to-v2.md)).
- **`StaticQuantity`** — value held as a hashable static field (for `jax.jit(static_argnames=...)`); equality is unit-label-based by design (`same_unit_label`), not physical equivalence.
- **`Angle`** — wrapping-aware `Quantity` subtype.

Dims (`unxt.dims`) parse expressions via a small AST-based grammar in `src/unxt/_src/dimensions.py` — `* / ** ()` are supported, unary `+`/`-` deliberately raise ("dimensions are invariant under negation," not a missing feature). Units (`unxt.units`) wrap `astropy.units`; `AbstractUnit = apyu.UnitBase | apyu.FunctionUnitBase` (`StructuredUnit` is deliberately excluded). Unit systems live under `src/unxt/_src/unitsystems/`.

Naming rule (see [docs/explanation/api-conventions.md](docs/explanation/api-conventions.md)): `Abstract...` prefix marks a non-instantiable base; no abstract class inherits from a concrete one, no concrete class inherits from another concrete one.

## The `_mk` unchecked constructor


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GalacticDynamics/unxt](https://github.com/GalacticDynamics/unxt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
