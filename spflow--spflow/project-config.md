---
trigger: always_on
description: - **Run all tests:** `.venv/bin/pytest -n 4`
---

# AI Agent Guide for SPFlow

## Development Commands
- **Run all tests:** `.venv/bin/pytest -n 4`
- **Run single test:** `.venv/bin/pytest tests/path/to/test_file.py::test_function`
- **Format code:** `.venv/bin/black spflow tests`
- **Build docs:** `cd docs && make html`
- **Generate HTML coverage report:** `.venv/bin/pytest -n 4 --cov=spflow --cov-report=html`
- **List lowest-coverage files:** `.venv/bin/python scripts/coverage_inspect.py list --limit 30`
- **Show missed line chunks (with context):** `.venv/bin/python scripts/coverage_inspect.py show spflow/path/to_file.py --context 3`
- **Coverage runtime note (PyTorch):** Avoid module-targeted `pytest-cov` like `--cov=spflow.learn.prometheus` in this environment; it can trigger `RuntimeError: function '_has_torch_function' already has a docstring` during `import torch`. Use package-level coverage targets instead (for example `--cov=spflow` or `--cov=spflow.learn`).

## Remote Execution with rr
- Use `rr` for heavy test runs, benchmarks, or any workload that should run on a remote machine instead of locally.
- **Check remote availability:** `rr status`
- **Run a command remotely:** `rr run "<command>"`
- **Run without syncing first:** `rr exec "<command>"`
- **Remote pytest example:** `rr run ".venv/bin/pytest tests/modules/test_factorize.py::test_sample_non_diff_runs_on_cuda -q"`
- **Remote benchmark example:** `rr run ".venv/bin/python scripts/profile_sampling_routing.py benchmark --workload factorize-int"`
- **Verify CUDA remotely before GPU-specific work:** `rr run ".venv/bin/python -c 'import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else \"no-cuda\")'"`
- Treat remote shell startup warnings as configuration issues, not benchmark results. If `rr` reaches the host but shell init breaks command execution, fix the host shell configuration before trusting failures.
- Prefer `rr` for remote CUDA regression tests so local CPU-only environments do not mask GPU-specific failures.

## Code Style Guidelines
- **Python version:** 3.10+ with type hints required
- **Docstrings:** Google style convention
- **Error handling:** Use custom exceptions from `spflow.exceptions`
- **PyTorch:** All Modules inherit from `nn.Module`, use proper tensor typing
- **Testing:** Use pytest, fixtures in `conftest.py`

## Project Structure
- Core modules in `spflow/modules/` with base classes in `base.py` files
- Leaf distributions in `spflow/modules/leaves/`
- Tests mirror source structure in `tests/`


## Versioning & Commits

* **Versioning:** Semantic Versioning. The version is in `spflow/__init__.py`.
* **Commits:** Use [Conventional Commits](https://www.conventionalcommits.org/). Keep the commit body brief. Don't mention which files changed in detail since we can see this in the git diff anyway.
* **NOTE:** Never `git add` or `git commit` unless I ask you to.

### Commit Message Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Commit Types

| Type       | Description                 | Version Bump |
|------------|-----------------------------|--------------|
| `feat`     | New feature                 | MINOR        |
| `fix`      | Bug fix                     | PATCH        |
| `docs`     | Documentation only          | -            |
| `style`    | Formatting, no logic change | -            |
| `refactor` | Code refactoring            | -            |
| `perf`     | Performance improvements    | -            |
| `test`     | Adding/updating tests       | -            |
| `chore`    | Maintenance tasks           | -            |
| `ci`       | CI/CD changes               | -            |
| `build`    | Build system changes        | -            |

### Scopes (Optional)

`modules`, `learn`, `leaf`, `rat`, `deps`, `tests`, `sum`, `product`

### Examples

```bash
git commit -m "feat(leaf): add Gamma distribution module"
git commit -m "fix(dispatch): correct cache initialization"
git commit -m "docs: update RAT-SPN examples in README"
```

## Programming Practices
* Prefer clarity, simplicity, and explicitness (Zen of Python).
* Write code that is correct, readable, maintainable, and efficient.
* Keep functions small and focused on one task (single responsibility).
* Keep modules cohesive; avoid unnecessary coupling.
* Prefer simple control flow; avoid deeply nested logic.
* Avoid repetition; follow DRY (Don’t Repeat Yourself).
* Never implement silent fallbacks when adding new features; fail fast with explicit errors or warnings.

## Comment Best Practices
* Follow PEP 8 / PEP 257 style for comments and docstrings, while keeping docstrings in Google style for public APIs.
* Use comments to explain intent, invariants, assumptions, side effects, and non-obvious tradeoffs; comment why this code exists, not what obvious syntax is doing.
* Prefer self-explanatory names and clear code; if a comment only paraphrases the code, rewrite the code instead.
* Use inline comments sparingly and only when the statement would otherwise be hard to understand; keep them separated from code and specific to the non-obvious detail.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SPFlow/SPFlow](https://github.com/SPFlow/SPFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
