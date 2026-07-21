---
trigger: always_on
description: **Cachier** is a Python library providing persistent, stale-free, local and cross-machine caching for Python functions via a decorator API. It supports multiple backends (pickle, memory, MongoDB, SQL, Redis), is thread-safe, and is designed for extensibility and robust cross-platform support.
---

# AGENTS.md

## 📦 Project Overview

**Cachier** is a Python library providing persistent, stale-free, local and cross-machine caching for Python functions via a decorator API. It supports multiple backends (pickle, memory, MongoDB, SQL, Redis), is thread-safe, and is designed for extensibility and robust cross-platform support.

- **Repository:** [python-cachier/cachier](https://github.com/python-cachier/cachier)
- **Primary Language:** Python 3.10+
- **Key Dependencies:** `portalocker`, `watchdog` (optional: `boto3`, `pymongo`, `sqlalchemy`, `redis`)
- **Test Framework:** `pytest` with backend-specific markers
- **Linting:** `ruff` (replaces black/flake8)
- **Type Checking:** `mypy`
- **CI:** GitHub Actions (matrix for backends/OS with Dockerized services)
- **Issue Tracking:** GitHub Issues
- **Additional Docs:** `.github/copilot-instructions.md` for contributor guidelines

______________________________________________________________________

## 🗂️ Repository Structure

```
cachier/
├── src/cachier/           # Main library code
│   ├── __init__.py
│   ├── core.py            # Decorator logic, backend selection
│   ├── cores/             # Backend implementations
│   │   ├── pickle.py
│   │   ├── memory.py
│   │   ├── mongo.py
│   │   ├── sql.py
│   │   ├── redis.py
│   │   ├── s3.py
│   │   └── base.py
│   ├── config.py          # Global/default config
│   ├── _types.py          # Type definitions
│   ├── _version.py
│   └── __main__.py
├── tests/                 # Pytest-based tests, backend-marked
│   ├── test_*.py
│   └── requirements_*.txt # Backend-specific test requirements
├── examples/              # Usage examples
├── README.rst             # Main documentation
├── pyproject.toml         # Build, lint, type, test config
├── .pre-commit-config.yaml
├── .github/               # CI, issue templates, workflows
└── ... (see full tree above)
```

______________________________________________________________________

## 🚦 Quick Start

1. **Install core dependencies:**

   ```bash
   pip install .[all]
   ```

   - For backend-specific dev: see `tests/requirements_*.txt`.

2. **Run tests:**

   ```bash
   pytest                           # All tests
   pytest -m "pickle or memory"     # Basic backends only
   pytest -m "not (mongo or redis or sql)"  # Exclude external service backends
   ```

3. **Lint and type-check:**

   ```bash
   ruff check .
   mypy src/cachier/
   ```

4. **Try an example:**

   ```bash
   # Quick test
   python -c "
   from cachier import cachier
   import datetime

   @cachier(stale_after=datetime.timedelta(days=1))
   def test_func(x):
       return x * 2

   print(test_func(5))  # Calculates and caches
   print(test_func(5))  # Returns from cache
   "

   # Or run the Redis example (requires Redis server)
   python examples/redis_example.py
   ```

______________________________________________________________________

## 🧑‍💻 Development Guidelines

### 1. **Code Style & Quality**

- **Python 3.10+** only.
- **Type annotations** required for all new code.
- **Docstrings:** Use numpy style, multi-line, no single-line docstrings.
- **Lint:** Run `ruff` before PRs. Use per-line/file ignores only for justified cases.
- **Type check:** Run `mypy` before PRs.
- **Testing:** All public methods must have at least one test. Use `pytest.mark.<backend>` for backend-specific tests.
- **No warnings/errors for missing optional dependencies at import time.** Only raise when backend is used.

### 2. **Backends**

- **Default:** Pickle (local file cache, `~/.cachier/`)
- **Others:** Memory, MongoDB, SQL, Redis, S3
- **Adding a backend:** Implement in `src/cachier/cores/`, subclass `BaseCore`, add tests with appropriate markers, update docs, and CI matrix if needed.
- **Optional dependencies:** Code/tests must gracefully skip if backend deps are missing. Install backend-specific deps via `tests/requirements_*.txt`.
- **Requirements files:** `tests/requirements_mongodb.txt`, `tests/requirements_postgres.txt`, `tests/requirements_redis.txt`, `tests/requirements_s3.txt` for backend-specific dependencies.

### 3. **Decorator Usage**

- Main API: `@cachier`
- Key params: `stale_after`, `backend`, `mongetter`, `cache_dir`, `pickle_reload`, `separate_files`, `wait_for_calc_timeout`, `allow_none`, `hash_func`
- Arguments to cached functions must be hashable. For unhashable, provide `hash_func`.

### 4. **Testing**

- **Run all tests:** `pytest`
- **Backend-specific:** Use markers, e.g. `pytest -m mongo`, `pytest -m redis`, `pytest -m sql`, `pytest -m s3`
- **Available markers:** `mongo`, `memory`, `pickle`, `redis`, `sql`, `s3`, `maxage` (see `pyproject.toml`)
- **Requirements:** See `tests/requirements_*.txt` for backend test deps.
- **CI:** Matrix covers OS/backend combinations. Mongo/SQL/Redis require Dockerized services.
- **Missing deps:** Tests gracefully skip if optional backend dependencies are missing.

### 5. **Documentation**

- **README.rst** is the canonical user/developer doc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [python-cachier/cachier](https://github.com/python-cachier/cachier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
