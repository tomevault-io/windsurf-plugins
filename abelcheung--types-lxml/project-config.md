---
trigger: always_on
description: - Help AI coding agents become productive quickly when editing stub files and test infra.
---

# Copilot instructions for types-lxml

## Purpose
- Help AI coding agents become productive quickly when editing stub files and test infra.

## Big picture
- This repository provides external type annotations for `lxml` under `src/lxml-stubs/`. Edit `.pyi` files in `src/lxml-stubs/` for type changes. Example: `src/lxml-stubs/etree/__init__.pyi` shows core `Element` APIs.
- Runtime test suite lives in `tests/runtime/` (with subfolders `elem/` for element tests and `html_/` for HTML-related tests), including:
  - `hypothesis` based property-based tests to cover broad input space.
  - Multi-version `lxml` testing to ensure compatibility across supported versions.
  - Multi-type-checker support (mypy, pyright, basedpyright, pyrefly) to ensure broad usability. Their outputs are captured and compared with runtime type checker (`typeguard`) results to ensure runtime correctness.
- Static tests live in `tests/static/`, which is intended to be phased out and replaced by runtime tests.
- A small `mypy` plugin is shipped at `src/mypy_plugin_lxml/main.py` to model `lxml` runtime behaviour for type checkers. Use `src/mypy_plugin_lxml/main.py` when you need behaviour that can't be expressed purely in stubs (e.g. special method hooks).

## Runtime tests
- Purpose: ensure the stubs match actual `lxml` runtime behavior and that type-checker feedback (revealtype/warnings) matches expectations when running against real `lxml` installs.
- Two complementary steps are used in CI and locally:
	- `mypy.stubtest` run against the installed wheel to detect runtime/static signature mismatches (see tox `_env_myst` commands in `pyproject.toml`). The allowlist `tests/runtime/allowlist.txt` suppresses known, intentional mismatches.
	- `pytest` runtime suite in `tests/runtime/` which executes behavior and type-checker adapters (via `pytest-revealtype-injector`) to capture reveal-type output across multiple checkers.
- Hypothesis: property-based testing is central — strategies are implemented in `tests/runtime/_testutils/strategy.py` and registered at collection time by `tests/runtime/register_strategy.py`. Many tests rely on generated names/elements/attributes to exercise a broad input space (XML names, attribute values, CDATA, processing-instructions, etc.). When editing or adding tests or stubs, reuse/extend these strategies rather than inventing new random generators.
- Test utilities & fixtures:
	- `tests/runtime/conftest.py` provides fixtures for sample files, `generate_input_file_arguments()` (yields many input forms: `Path`, `bytes`, file-like, compressed streams, `urlopen`), and light-weight fixtures (`disposable_element`, `disposable_attrib`) to keep Hypothesis tests fast.
	- `tests/runtime/_testutils/` contains helpers used throughout tests: `strategy.py` (Hypothesis strategies), `decorator.py` (signature testers used to assert runtime C extension signatures), `common.py` (type buckets and helpers), `errors.py` (prebuilt pytest.raises contexts), and `marker.py` (xfail markers for known runtime bugs).
- Why this design: running tests against multiple real `lxml` versions and multiple type checkers (mypy, pyright, basedpyright, pyrefly) gives stronger guarantees than static-only checks. Hypothesis generates edge cases that human-written examples often miss; extensive fixtures exercise IO variants and parser options to validate union types and overloads in stubs.

### reveal_type() and `pytest-revealtype-injector`
- The runtime suite uses `pytest-revealtype-injector` to turn plain `reveal_type()` calls in tests into coordinated static+dynamic checks. The plugin (installed in CI via test dependencies) does three things:
	- Replaces global `reveal_type()` calls at collection time so tests can call `typing.reveal_type(x)` with no boilerplate.
	- Runs configured static type checkers (mypy, pyright, basedpyright, pyrefly) to collect each checker's `reveal_type` output for the code under test.
	- Uses `typeguard` at runtime to assert that the static reveal results actually match the runtime types (failing the test when a mismatch is found).
- How tests use it: tests simply import or call `reveal_type` (module-level import) and assert or examine its return. The plugin scans tests for global imports and single-line `reveal_type()` calls (see `tests/runtime/register_strategy.py` and `tests/runtime/conftest.py` for registration and plugin activation).
- Disabling and markers: tests can opt-out individual checkers using `@pytest.mark.notypechecker("mypy")` (function/class/module level) if a given checker should be excluded for a test case.
- Logging and debug: the plugin logs replacements and extraction results; run `pytest -v` to see `INFO`/`DEBUG` logs that include extracted `reveal_type` code and checker outputs. This is useful when a test fails because a static checker disagrees with runtime behavior.


## Key project conventions
- Prefer explicit `.pyi` signatures and annotation-specific docstrings.
- There are two builds: the default and `types-lxml-multi-subclass` (see README). Maintainer would handle the latter build manually, there is no need to take care of it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abelcheung/types-lxml](https://github.com/abelcheung/types-lxml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
