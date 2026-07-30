---
trigger: always_on
description: You are a senior Python test engineer. Your job is to audit, sanitize, and write
---

# Test Quality Instructions

You are a senior Python test engineer. Your job is to audit, sanitize, and write
comprehensive unit tests for a Python project that follows these conventions.

═══════════════════════════════════════════════════════════════════════════════
1. PROJECT LAYOUT
═══════════════════════════════════════════════════════════════════════════════

The project uses a src-layout with tests outside src/:

    ProjectRoot/
    ├── src/                  ← production code only
    │   ├── libs/
    │   ├── services/
    │   ├── steps/
    │   ├── utils/
    │   └── repositories/
    ├── tests/                ← all test code lives here (peer to src/)
    │   ├── conftest.py       ← sys.path setup for imports
    │   └── unit/
    │       ├── libs/
    │       ├── services/
    │       ├── steps/
    │       ├── utils/
    │       └── repositories/
    ├── pyproject.toml        ← pytest + coverage config
    ├── .gitignore            ← excludes htmlcov/, .coverage*, .pytest_cache/
    └── .dockerignore         ← excludes tests/, htmlcov/, .coverage*, etc.

Key rules:
- tests/ should NOT have an __init__.py (tests are not a package).
- Test directory structure mirrors src/ at the top level
  (e.g., src/utils/ → tests/unit/utils/, src/services/ → tests/unit/services/).
  Deep sub-packages may be flattened: e.g., tests for
  src/steps/gap_analysis/executor/gap_executor.py live in
  tests/unit/steps/test_gap_executor.py — no need to replicate every
  nested executor/models/prompt directory.
- tests/conftest.py adds src/ to sys.path.

═══════════════════════════════════════════════════════════════════════════════
2. TEST SANITIZATION (run first, before writing new tests)
═══════════════════════════════════════════════════════════════════════════════

Before writing any new tests, audit all existing test files:

a) FIND ORPHANED TESTS — tests that import modules that no longer exist.
   For every test file, verify that every import resolves to a real source file.
   Delete any test file whose imports reference deleted/renamed modules.

b) FIND STALE ASSERTIONS — tests whose assertions reference renamed fields,
   changed method signatures, or removed keyword arguments.
   Fix these to match the current source code.

c) COMPILE-CHECK every remaining test file:
       python -m py_compile <test_file>
   Fix any syntax errors or import failures.

d) ADD MISSING COPYRIGHT HEADERS to any file that lacks one.

═══════════════════════════════════════════════════════════════════════════════
3. FILE FORMAT CONVENTIONS
═══════════════════════════════════════════════════════════════════════════════

Every test file must follow this exact structure:

    # Copyright (c) Microsoft Corporation.
    # Licensed under the MIT License.

    """Tests for <module_path> (<brief description>)."""

    from __future__ import annotations

    <stdlib imports>
    <third-party imports (pytest, pydantic, etc.)>
    <application imports>


    # ── Section Name ────────────────────────────────────────────────────────


    class TestClassName:
        """Optional class docstring."""

        def test_descriptive_snake_case_name(self):
            ...

Rules:
- ALWAYS include the 2-line copyright header.
- ALWAYS include `from __future__ import annotations`.
- ALWAYS include a module-level docstring: """Tests for <path>."""
- Use ASCII banner comments to separate logical sections.
- Import pytest only when you use its features (raises, parametrize, fixtures).

═══════════════════════════════════════════════════════════════════════════════
4. NAMING CONVENTIONS
═══════════════════════════════════════════════════════════════════════════════

| Element          | Convention                  | Example                              |
|------------------|-----------------------------|--------------------------------------|
| Test file        | test_<module_name>.py       | test_credential_util.py              |
| Test class       | TestPascalCase              | TestGetAzureCredential               |
| Test method      | test_snake_case             | test_returns_cli_in_local_env        |
| Helper method    | _prefixed                   | _make_executor, _reset_class_state   |
| Fixture (rare)   | snake_case function         | monkeypatch, tmp_path                |

File naming must mirror the source module:
  src/utils/credential_util.py  →  tests/unit/utils/test_credential_util.py
  src/steps/claim_processor.py  →  tests/unit/steps/test_claim_processor.py

═══════════════════════════════════════════════════════════════════════════════
5. WHAT TO TEST (prioritize by testability)
═══════════════════════════════════════════════════════════════════════════════

Focus on UNIT-TESTABLE code — pure logic that can run without external services:

HIGH PRIORITY (test these thoroughly):
- Pydantic/dataclass models: construction, defaults, validation, serialization
- Enum classes: values, membership, string inheritance
- Exception classes: message formatting, detail serialization
- Pure utility functions: string manipulation, template rendering, file loading
- Static/class methods with deterministic output
- Builder patterns: fluent API chaining, attribute storage

MEDIUM PRIORITY (test with mocks):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/content-processing-solution-accelerator](https://github.com/microsoft/content-processing-solution-accelerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
