---
trigger: always_on
description: Operational notes for agents working in this repo. Read before making changes.
---

# AGENTS.md

Operational notes for agents working in this repo. Read before making changes.

## Toolchain

- Python is pinned exactly to `3.12.10` (`requires-python = "==3.12.10"`). Not `>=3.12`. That applies to the engine; `bootstrap/` (published as `mship`) is `>=3.10` because it runs before the pinned environment exists.
- Dependency manager is **uv**.
- Never run `pip install`; always use `uv sync` / `uv run` / `uv lock`.
- `cuda` and `cpu` extras are mutually exclusive (declared in `[tool.uv] conflicts`). `torch` / `torchvision` come from different indexes per extra (`pytorch-cu130` vs `pytorch-cpu`). A third extra, `thin`, is empty (base deps only) — no torch/vllm, used by the thin control/coordinator image.

## Commands you'd otherwise guess wrong

```bash
# Install deps for development (choose cuda OR cpu, plus dev)
uv sync --extra dev --extra cuda   # what CI uses
uv sync --extra dev --extra cpu --extra vllm-cpu   # CPU-only dev (vllm-cpu pulls `openai`, which conftest needs)

# The canonical dev loop (mirrored in CI and Makefile)
make lint        # ruff check + ruff format --check + pyright  (all three MUST pass)
make lint-fix    # ruff check --fix + ruff format
make test        # uv run pytest tests/ -v

# Run a single test
uv run pytest tests/test_config.py::TestLlamaServerConfig::test_defaults -v
```

CI (`.github/workflows/ci.yml`) runs `uv sync --extra dev --extra cuda` on Linux, then `ruff check`, `ruff format --check`, `pyright`, and `pytest tests/ -v -m "not integration"` — same filter as the "skip integration" guidance below. Separate CI jobs cover the `bootstrap/` package (multi-Python-version matrix), lockfile/pins parity, and the Helm chart. Match the lint+test job locally before pushing.

`make lint` requires `--extra cuda` to be installed. Pyright resolves imports against the active venv, and `gguf`, `diffusers`, and `psutil` only ship under the cuda extra, so lint on a cpu-only sync fails with `reportMissingImports`. (`vllm` is importable under both extras as of the Stage E0 CPU wheel wiring — it's no longer cuda-only, just not enough on its own to make lint pass cpu-only.) Tests run fine on either extra (the cuda extra is a superset).

Agents: when running tests on your own initiative (sanity-checking a change, verifying a bump), skip the slow `integration`-marked suite by default — `uv run pytest tests/ -v -m "not integration"`. Only run the full `make test` (which includes integration) when explicitly requested.

Pre-commit only runs ruff; it does **not** run pyright or tests, so don't rely on the hook to catch type errors.

## OpenAI protocol fidelity

`modelship/openai/protocol.py` is the request/response surface clients see. When adding or changing models there:

- **Follow the official OpenAI API specification strictly.** Field names, types, defaults, optionality, and shape of nested objects must match what `platform.openai.com/docs` documents for the corresponding route.
- Do not invent fields to expose loader-specific knobs (Diffusers `strength`, vLLM `stop_reason`, etc.). Carry loader-specific defaults via the per-model `*_config` in `infer_config.py` instead.
- Missing optional OpenAI fields are fine when a feature is genuinely unsupported. Adding fields that aren't in OpenAI's spec is not — it locks clients into a modelship-specific dialect and breaks the drop-in-replacement guarantee.
- When OpenAI's spec evolves (new fields, new response_format values, new routes), update the protocol shapes before wiring the backend.

When in doubt, check OpenAI's reference for the exact route. Existing deviations are documented and tracked separately; do not add new ones.

## Lint / format / typecheck rules

- Line length **120** (not 88). Ruff handles formatting; `E501` is disabled because the formatter owns line length.
- Ruff rule set: `E, W, F, I, N, UP, B, SIM, RUF`. `I` means isort runs — don't hand-sort imports.
- Pyright `typeCheckingMode = "basic"`, scoped to `modelship`, `mship_deploy.py`. Don't add `# type: ignore` without checking pyright actually complains in basic mode.

## Running the server

`mship deploy` (console script, installed via `pip`/`uv tool install "mship[metal]"`) is the entry point; `modelship/launcher.py` resolves the cache root, checks the Python version, detects the accelerator (`cuda`/`rocm`/`xpu`/`metal`/`cpu`, keyed on the installed torch build — see `modelship/utils/accelerator.py`), and on macOS auto-provisions `llama-server` before handing off to `modelship/driver.py:main`. `mship_deploy.py` survives as a 3-line back-compat shim to `modelship.driver.main`, for source runs. The driver itself:

1. Reads `config/models.yaml` (gitignored — copy one from `config/examples/`). An explicit `--config <path>` that doesn't exist is a hard error; absent both `--config` and the default file, it bootstraps an empty coordinator (gateway up, no models) that waits for a later `--config`/`--reconcile` or a joining node.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modelship-ai/modelship](https://github.com/modelship-ai/modelship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
