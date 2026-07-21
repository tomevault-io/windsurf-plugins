---
trigger: always_on
description: SPDX-FileCopyrightText: 2019–2026 Pynguin Contributors
---

<!--
SPDX-FileCopyrightText: 2019–2026 Pynguin Contributors

SPDX-License-Identifier: CC-BY-4.0
-->

<!-- Generated: 2026-01-30 | Updated: 2026-01-30 -->

# Pynguin - Automated Unit Test Generation for Python

## Purpose

Pynguin (IPA: ˈpɪŋɡuiːn) is a research tool for automated generation of unit tests for Python programs. It uses search-based techniques (genetic algorithms) to automatically generate test suites that achieve high code coverage. The tool is developed at the University of Passau and represents the first fully-automated unit test generator for general-purpose Python programs.

## Project Status

- **Python Support:** 3.10 (stable), 3.11-3.14 (experimental)
- **License:** MIT

## Key Files

| File | Description |
|------|-------------|
| `pyproject.toml` | Poetry project configuration, dependencies, and tool settings |
| `README.md` | Project overview, installation, and usage instructions |
| `Makefile` | Development commands (check, test, coverage, docs) |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `src/` | Main source code implementation (see `src/AGENTS.md`) |
| `tests/` | Comprehensive test suite with fixtures (see `tests/AGENTS.md`) |
| `docs/` | Sphinx documentation (user guides, API reference) (see `docs/AGENTS.md`) |
| `docker/` | Docker configurations for isolated execution (see `docker/AGENTS.md`) |
| `.run/` | PyCharm run configurations |
| `.idea/` | PyCharm IDE settings |
| `LICENSES/` | License texts (MIT, Apache-2.0, BSD-2-Clause, CC-BY-4.0, CC0-1.0) |

## For AI Agents

### Working In This Project

1. **Development Setup:**
   - Use Poetry for dependency management: `poetry install`
   - Python 3.10+ required
   - Run `make check` before committing (runs tests, linting, type checking)

2. **Code Quality Standards:**
   - **Linting:** Ruff (extensive ruleset enabled, see pyproject.toml)
   - **Type Checking:** mypy with strict settings
   - **Formatting:** Black formatter
   - **Documentation:** Google-style docstrings
   - **Testing:** pytest with >80% coverage requirement

### Testing Requirements

- **Unit Tests:** Located in `tests/` mirroring `src/` structure
- **Coverage:** Minimum 80%, configured in `tool.coverage` section
- **Test Framework:** pytest with plugins (pytest-cov, pytest-mock, pytest-sugar)
- **Run Tests:** `make test` or `pytest tests/`
- **Coverage Report:** `make cov` generates HTML report in `cov_html/`

### Common Patterns

1. **Module Structure:**
   - Source code in `src/pynguin/`
   - Tests in `tests/` with same directory structure
   - Fixtures in `tests/fixtures/` organized by feature area

2. **Genetic Algorithm Architecture:**
   - Algorithms in `src/pynguin/ga/algorithms/`
   - Operators (crossover, selection, mutation) in `src/pynguin/ga/operators/`
   - Test case representation in `src/pynguin/testcase/`

3. **Instrumentation:**
   - Code instrumentation in `src/pynguin/instrumentation/`
   - Version-specific handling in `src/pynguin/instrumentation/version/`
   - Instrumentation transformations for coverage tracking

4. **Type System:**
   - Type tracing and inference throughout codebase
   - Active optimization work to reduce execution overhead
   - Fixtures in `tests/fixtures/type_tracing/`

5. **LLM Integration (Optional):**
   - Large language model features in `src/pynguin/large_language_model/`
   - Requires `openai` extra: `poetry install --extras openai`
   - Assertion generation and test case prompting

## Dependencies

### Core Dependencies

- **astroid** - Python AST analysis
- **bytecode** - Python bytecode manipulation
- **networkx** - Graph algorithms (for control flow)
- **pytest** - Testing framework (used both for running Pynguin and generated tests)
- **black** - Code formatting
- **libcst** - Concrete syntax tree manipulation
- **Jinja2** - Template engine (for test generation)

### Optional Features ("--extras")

| Extra | Dependencies | Purpose |
|-------|--------------|---------|
| `openai` | openai, python-dotenv | LLM-based test generation |
| `numpy` | numpy | Numerical computation support |
| `typing` | mypy, typing-extensions | Enhanced type checking |
| `fandango-faker` | faker, fandango-fuzzer, xmltodict | Resource generation (test data) |

### Development Dependencies

- **mypy** - Static type checking
- **ruff** - Fast Python linter
- **pre-commit** - Git hooks for code quality
- **sphinx** - Documentation generation
- **coverage** - Code coverage measurement
- **hypothesis** - Property-based testing

## Architecture Overview

```
Pynguin Test Generation Pipeline:
1. Module Analysis → Parse Python code, build control flow graph
2. Algorithm Selection → Choose search strategy (DYNAMOSA, MIO, MOSA, etc.)
3. Test Generation → Genetic algorithm evolves test cases
4. Assertion Generation → Add assertions via mutation analysis or LLM
5. Output → Generated test suite as executable pytest tests
```

### Key Components

- **GA (Genetic Algorithms):** Core search-based test generation
- **Instrumentation:** Runtime code monitoring for coverage tracking
- **Assertion Generation:** Automated oracle creation
- **Test Case Representation:** Internal model for test cases
- **Slicer:** Program slicing for focused testing
- **Resources:** Test data generation (primitives, objects, collections)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [se2p/pynguin](https://github.com/se2p/pynguin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
