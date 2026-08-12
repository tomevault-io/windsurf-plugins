---
trigger: always_on
description: Professional-grade LLM benchmark for embedded firmware development (Zephyr RTOS, FreeRTOS, ESP-IDF, STM32 HAL, Linux kernel drivers, Yocto/Embedded Linux).
---

# EmbedEval

Professional-grade LLM benchmark for embedded firmware development (Zephyr RTOS, FreeRTOS, ESP-IDF, STM32 HAL, Linux kernel drivers, Yocto/Embedded Linux).

## Project Context

- **Vault Project:** embedeval
- **Repository:** /home/noel/embedeval
- **TODO Sync:** Enabled
- **GitHub:** https://github.com/Ecro/embedeval
- **Private Cases:** https://github.com/Ecro/embedeval-private (48 held-out TCs)

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Python | 3.12+ |
| Package Manager | uv | latest |
| LLM Client | litellm | latest |
| CLI | typer | latest |
| Models | pydantic | v2 |
| Container | Docker | latest |
| Target SDKs | Zephyr RTOS, FreeRTOS, ESP-IDF, STM32 HAL, Linux kernel, Yocto | various |
| CI | GitHub Actions | - |

## Project Structure

```
embedeval/
├── src/embedeval/       # Core library
│   ├── models.py        # Pydantic models (EvalResult, BenchmarkReport, etc.)
│   ├── evaluator.py     # 5-layer evaluation pipeline (L0-L4)
│   ├── runner.py        # Benchmark runner orchestration
│   ├── scorer.py        # Score aggregation (pass@1, pass@5, layer rates)
│   ├── reporter.py      # JSON + Markdown report generation
│   ├── llm_client.py    # LiteLLM wrapper with retry logic
│   └── cli.py           # Typer CLI (run, list, validate, report)
├── cases/               # Public test cases (179 TCs)
│   ├── kconfig-001/
│   ├── device-tree-001/
│   └── isr-concurrency-001/
│                        # Private cases in separate repo: ../embedeval-private/cases/
├── tests/               # pytest test suite
├── docs/                # METHODOLOGY.md, CONTRIBUTING.md
└── .github/workflows/   # CI, case validation, benchmark dispatch
```

## Commands

| Command | Purpose |
|---------|---------|
| `uv run pytest` | Run all tests |
| `uv run ruff check src/ tests/` | Lint |
| `uv run ruff format src/ tests/` | Format |
| `uv run mypy src/` | Type check |
| `uv run embedeval --help` | CLI help |
| `uv run embedeval list --cases cases/` | List cases |
| `uv run embedeval validate --cases cases/` | Validate cases |
| `uv run embedeval run ... --private-cases ../embedeval-private/cases/ --include-private` | Include private cases |

## Available Workflow Commands

- `/research` - Gather best practices, explore options
- `/myplan [task]` - Analyze, design, and plan implementation
- `/execute [task]` - Implement and test
- `/review [task]` - Code review and quality check
- `/wrapup [task]` - Finalize, commit, PR, and complete

## Documentation Auto-Sync (MANDATORY)

**At `/wrapup` or before any commit that changes `cases/`, `src/`, or `tests/`:**

```bash
uv run python scripts/sync_docs.py
```

This auto-updates:
- `docs/METHODOLOGY.md` — TC count, platform/difficulty distribution, negatives count
- `README.md` — test count, module count, case count, insights count

**Always commit the updated docs together with code changes.**

If `sync_docs.py` output shows "already up to date", no action needed.

## 23 Evaluation Categories

Platform-agnostic: `gpio-basic`, `uart`, `adc`, `pwm`, `spi-i2c`, `dma`, `isr-concurrency`, `threading`, `timer`, `sensor-driver`, `networking`, `ble`, `security`, `storage`

System-level: `kconfig`, `device-tree`, `boot`, `ota`, `power-mgmt`, `watchdog`

Platform-specific: `yocto`, `linux-driver`, `memory-opt`

## 5-Layer Evaluation Architecture

- **L0 Static**: Pattern matching, required includes, structure checks
- **L1 Compile**: Docker-based SDK compilation (west build, idf.py, make)
- **L2 Runtime**: QEMU/native_sim execution with timeout
- **L3 Behavioral**: Output validation against expected patterns
- **L4 Mutation**: Robustness testing with code mutations

## Boundaries

- NO placeholder code or TODO comments
- NO skipping tests
- NO adding dependencies without justification
- All new test cases must include reference solutions that pass all layers

## Quality Gates (MANDATORY before commit)

1. **Self-review `git diff`** — Read every changed line before committing. Check:
   - API contracts: does the function I'm calling return what I expect?
   - Redundant calls: am I calling a function that was already called upstream?
   - Edge cases: what happens with empty input, None, malformed data?

2. **New feature = new tests** — Every new CLI option, module, or code path needs at least 3 tests:
   - Happy path
   - Edge case (empty, None, boundary)
   - Error case (invalid input, timeout)

3. **Check function signatures before calling** — Read the function definition before using it.
   Don't assume what it returns. Don't wrap it in "defensive" double-processing.

## Pre-commit Hook (one-time setup per clone)

CI's Lint job (`ruff format --check src/` + `ruff check src/`) is mirrored in
`.githooks/pre-commit`. Activate it once per clone:

```bash
git config core.hooksPath .githooks
```

`core.hooksPath` is a local-only git config, so it is **not** inherited on clone
— every fresh clone must run the command above, or CI will keep catching
format drift after push.

## Learned Corrections

### 2026

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ecro/embedeval](https://github.com/Ecro/embedeval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
