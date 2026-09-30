---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

## Environment

Dependencies are managed with `uv` (`uv.lock` is committed). Dev tooling lives in PEP 735 dependency groups, not extras, so the `pip install -e '.[dev,...]'` line in `Makefile`/`CONTRIBUTING.md` is out of date. Mirror CI instead:

```sh
uv sync --extra cpu --group dev --group test            # core development
uv sync --extra cpu --group dev --group test --group docs --group examples   # + docs / examples
```

Prefix commands with `uv run` (or activate `.venv`). `funsor` is pulled from git (`[tool.uv.sources]`).

## Commands

```sh
make lint      # ruff check, ruff format --check, license-header check, ty check
make format    # adds license headers, ruff format, ruff check --fix
make test      # lint, then pytest -v test  (slow: the full suite)
make docs      # sphinx html -> docs/build/html (needs pandoc)
make doctest   # docstring tests via sphinx, forced to CPU
python -m doctest -v README.md   # README snippets are tested in CI
```

Single test / subset:

```sh
pytest -vs test/test_distributions.py::test_log_prob -k Gompertz
pytest -vs -n auto test/infer/test_svi.py        # needs pytest-xdist
```

Variants CI runs, which matter when touching the corresponding code:

```sh
JAX_ENABLE_X64=1 pytest -vs test/infer/test_mcmc.py -k x64                      # double precision
XLA_FLAGS="--xla_force_host_platform_device_count=2" pytest -vs test/infer/test_mcmc.py -k "chain or pmap or vmap"   # multi-chain
JAX_ENABLE_CUSTOM_PRNG=1 pytest -vs test/infer/test_mcmc.py                     # typed PRNG keys
JAX_CHECK_TRACER_LEAKS=1 pytest -vs test/infer/test_mcmc.py::test_chain_inside_jit
CI=1 XLA_FLAGS="--xla_force_host_platform_device_count=2" pytest -vs -k test_example   # runs examples/ scripts
```

CI shards the suite as: everything except `test/infer` and `test/contrib` with `-k "not test_example"` (modeling); `test/contrib` + `test/infer` (inference); `-k test_example` (examples). `test/contrib/test_nested_sampling.py` requires `JAX_ENABLE_X64=1`.

Benchmarks (compared against the merge base on PRs): `python -m benchmarks.runner --list`, `NUMPYRO_BENCH_QUICK=1 python -m benchmarks.runner --suite handlers -o smoke.json`. See `benchmarks/README.md`.

## Things that will fail CI

- **Warnings are errors.** `pyproject.toml` sets pytest `filterwarnings = ["error", ...]`; a new `DeprecationWarning`/`UserWarning` on a tested path fails the test. Use `pytest.warns` or fix the source.
- **License headers.** Every non-empty `.py` file must begin with the two-line Pyro copyright / `SPDX-License-Identifier: Apache-2.0` header. `make license` (or `make format`) adds it.
- **Type annotations are enforced selectively.** Ruff's `ANN` rules apply only to the typed modules listed in `[tool.ruff.lint.per-file-ignores]` (`diagnostics.py`, `handlers.py`, `optim.py`, `patch.py`, `primitives.py`, `infer/elbo.py`, `distributions/distribution.py`), and `ty check` covers the include list in `[tool.ty.src]` (all of `numpyro/distributions`, most of `numpyro/infer`, several contrib packages). Code in those paths must be annotated and type-check; shared aliases are in `numpyro/_typing.py`.
- **Import order** is ruff-isort with a custom `known-jax` section (`flax`, `jax`, `optax`, `tensorflow_probability`) placed between third-party and first-party, `force-sort-within-sections`. Let `make format` do it.
- pre-commit additionally runs `codespell`, `yamlfmt`, and `tombi` (TOML formatting) — `pyproject.toml` edits must stay tombi-formatted.
- `test/conftest.py` forces the CPU platform, reseeds with `set_rng_seed(0)` before every test, enables x64 when `JAX_ENABLE_X64` is set, and asserts `jax.live_arrays()` is empty before the first test — so test modules must not create JAX arrays at import/collection time (build parametrize data with NumPy).

## Architecture

NumPyro is Pyro's modeling API re-implemented on JAX. Three layers, each usable independently:

### 1. Primitives + effect handlers (`primitives.py`, `handlers.py`)

A model is a plain Python function calling `numpyro.sample`, `param`, `deterministic`, `plate`, `factor`, etc. Each primitive builds a **message dict** (`type`, `name`, `fn`, `args`, `kwargs`, `value`, `is_observed`, `scale`, `mask`, `cond_indep_stack`, `infer`, ...) and passes it through `apply_stack` over the global `_PYRO_STACK` of `Messenger`s: `process_message` runs from the top of the stack down, then the default sampler runs if no handler set `value`, then `postprocess_message` runs back up. With an empty stack, `sample` just calls the distribution (which is why a `rng_key` is then required).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pyro-ppl/numpyro](https://github.com/pyro-ppl/numpyro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
