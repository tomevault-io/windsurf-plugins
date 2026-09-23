---
trigger: always_on
description: Notes for AI coding agents working in this repository. Humans may find them
---

# AGENTS.md

Notes for AI coding agents working in this repository. Humans may find them
useful too. Conventions follow <https://agents.md>.

## What this project is

A PyTorch implementation of two Bayesian changepoint detection algorithms:

- **Offline** (`offline_changepoint_detection`): Fearnhead (2006), posterior
  over changepoint locations via dynamic programming over segments. Exact:
  the sum over segment ends is evaluated in full. The `truncate` argument
  (default `-inf`) only exists to reproduce the truncated results of
  versions up to 1.1.0; that rule could discard the dominant term (see
  the docstring) and saved no work once the recursion was vectorized.
- **Online** (`online_changepoint_detection`): Adams & MacKay (2007), a
  recursively updated posterior over *run length* (time since the last
  changepoint).

It is a small research library. Prefer clarity and numerical correctness over
cleverness, and keep the public API stable.

## Language

Write this file, all other agent instructions, every identifier
(function, method, class, argument, test marker) and docstrings in American
English:
`initialize`, `normalize`, `behavior`, `color`, `neighbor`, `modeling`.
The rule is mandatory. Renaming a public identifier to comply needs a
deprecation path (keep the old name as an alias that warns, note it in
`CHANGELOG.md`); an audit on 2026-09-22 found no public name that needed
it, and renamed the test marker `behaviour` to `behavior`.

## Setup and tests

```bash
pip install -e ".[dev]"
pytest
```

CI lints with ruff (`ruff check` and `ruff format --check`, config in
`pyproject.toml`); run both before pushing, or `pre-commit install` once.
Tests must pass without a GPU.

Every test is marked `math` (checked against an independent computation
of the same quantity: scipy, an exhaustive enumeration, a closed form from
a paper, a derivation sharing no code with the one under test) or
`behavior` (pins current behavior: contracts, edge cases, devices, one
formula through two code paths, synthetic-data detection, goldens from an
earlier version); `tests/conftest.py` rejects a test with neither or both. `pytest -m math`
runs only the proofs, which is the set to watch when changing the
mathematics (see "Changing the math").

Two things that surprise people:

- **The suite is dramatically slower on an accelerator**: small tensors on
  MPS or CUDA are slower than on the CPU (minutes against seconds on Apple
  Silicon). The default device is the CPU since 1.2.0 (`get_device(None)`;
  `"auto"` opts into an accelerator), and both detectors follow the
  likelihood's device. Still pass `device="cpu"` explicitly in tests unless
  the test is specifically about device handling.
- Tests that need a GPU should carry `@pytest.mark.gpu` and skip themselves
  when none is present. Most do; `test_device_consistency` in
  `tests/test_integration.py` skips without the marker. Do not make a test
  depend on an accelerator being present without a skip.

## Layout

| Path | Contents |
| --- | --- |
| `bayesian_changepoint_detection/bayesian_models.py` | Both detection algorithms |
| `bayesian_changepoint_detection/segments.py` | `segment_statistics`: per-segment summaries and the direction of each change |
| `bayesian_changepoint_detection/streaming.py` | `OnlineChangepointDetector`, the online recursion one observation at a time |
| `bayesian_changepoint_detection/offline_likelihoods.py` | Segment likelihoods for the offline algorithm |
| `bayesian_changepoint_detection/online_likelihoods.py` | Predictive likelihoods for the online algorithm |
| `bayesian_changepoint_detection/priors.py` | Segment-length priors (offline) |
| `bayesian_changepoint_detection/hazard_functions.py` | Hazard functions (online) |
| `bayesian_changepoint_detection/device.py` | Device selection and tensor coercion |
| `bayesian_changepoint_detection/generate_data.py` | Synthetic series for tests and examples |
| `tests/` | Test suite |
| `benchmarks/` | Timings across released versions (`performance.py`) and detection quality on TCPD (`tcpd.py`); protocols in its README, results in `benchmarks/results/` |
| `examples/` | Runnable scripts and notebooks |
| `docs/`, `mkdocs.yml` | Documentation site; most pages include `README.md` sections through `<!-- --8<-- [start:name] -->` markers, so keep those markers intact |

## Things that are easy to get wrong

**There are two different classes each named `BaseLikelihood`, `StudentT`,
`MultivariateT`, `Poisson` and `NormalKnownVariance`.** One of each is in `offline_likelihoods`,
the other in `online_likelihoods`. They are unrelated and their interfaces are
incompatible. Always import them module-qualified
(`offline_likelihoods.StudentT`), never bare into a shared namespace.

**The offline and online likelihood interfaces differ:**

- Offline: `pdf(data, t, s)` returns the log likelihood of the *segment*
  `data[t:s]` under the model's prior — a single scalar. `s` is
  **exclusive**. (With #50 merged this is the exact marginal likelihood;
  before it, `StudentT` scored each point under the posterior of the whole
  segment, an approximation.)
- Online: `pdf(data)` takes one observation and returns a **vector** of log

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hildensia/bayesian_changepoint_detection](https://github.com/hildensia/bayesian_changepoint_detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
