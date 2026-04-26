---
trigger: always_on
description: These goals are what we are working towards in this project:
---

# Goals

These goals are what we are working towards in this project:
- A complete, widely compatible Prolog implementation
- Robust: It should handle all kinds of error conditions gracefully
- Fast: It should run prolog programs fast
- Scalable: It should be able to run large prolog programs

# Coding rules
- Follow PEP 8 for Python code
  - **All imports must be at the top of the file** (after module docstring, before other code)
  - Never use `import` statements inside functions or methods
  - Group imports: standard library, third-party, local (separated by blank lines)
- Run all tests before pushing a PR
- Use type hints where helpful
- Add docstrings to all public methods
- Keep functions focused and testable
- Use descriptive variable names



# Prolog Interpreter - Development Guide

This document provides an overview of the project structure and guidelines for development and testing.

## Project Structure

See @docs/ARCHITECTURE.md

## Features

See @docs/FEATURES.md

You must keep ./docs/FEATURES.md up to date when you add or change these

## Documentation

**All new markdown documentation should go in @docs/** 

Standard documentation files:
- `@docs/ARCHITECTURE.md` - System architecture and design
- `@docs/FEATURES.md` - ISO Prolog conformance matrix and feature checklist
- Any additional documentation, guides, or specs should be added as new files within the `@docs/` directory.

## Error Handling 

Handle Prolog errors like this:
```python
   except PrologThrow:
      # Re-raise Prolog errors
      raise
   except Exception: # Or use a more specific Exception if possible
      # Unexpected errors - convert to evaluation error
      raise PrologThrow(PrologError.evaluation_error("error", f"{op}/2"))
```



## Testing Guidelines

### Where to Put Tests

**All test files must go in the `tests/` directory.**

- **Unit tests**: `test_<module>.py` - Tests for specific modules
- **Integration tests**: `test_<feature>.py` - Tests for features spanning multiple modules
- **Fixtures**: `*.pl` files - Prolog code used by tests


### Test File Naming

- Python test files: `test_*.py`
- Prolog fixtures: `test_*.pl` or descriptive names like `minimal_mi_test.pl`
- Follow pytest conventions


### Writing Tests

1. **Import the interpreter**:
   ```python
   from vibeprolog import PrologInterpreter
   ```

2. **Create a new interpreter for each test**:
   ```python
   def test_something():
       prolog = PrologInterpreter()
       # Your test code
   ```

3. **Use pytest test classes to organize related tests**:
   ```python
   class TestAppend:
       """Tests for append/3 predicate."""

       def test_append_two_lists(self):
           prolog = PrologInterpreter()
           result = prolog.query_once("append([1, 2], [3, 4], X).")
           assert result['X'] == [1, 2, 3, 4]
   ```

4. **Test both success and failure cases**:
   ```python
   def test_unification_success(self):
       prolog = PrologInterpreter()
       assert prolog.has_solution("X = 5")

   def test_unification_failure(self):
       prolog = PrologInterpreter()
       assert not prolog.has_solution("1 = 2")
   ```


### Running Tests

```bash
# Run all tests
uv run pytest 

# Run all tests including performance (skipped by default). This is slow so run only when required.
uv run pytest --run-performance

# Run all slow tests (tests empirically longer than ~4 seconds). Skip unless flag is present.
uv run pytest --run-slow-tests

# Run specific test file
uv run pytest tests/test_builtins.py

# Run specific test class
uv run pytest tests/test_builtins.py::TestAppend

# Run specific test
uv run pytest tests/test_builtins.py::TestAppend::test_append_two_lists

# Run with verbose output
uv run pytest -v

# Run with output capture disabled (see print statements)
uv run pytest -s
```

### Long-running tests

- When adding a new test that consistently takes longer than ~4 seconds, mark it with `@pytest.mark.performance` (or set `pytestmark = pytest.mark.performance` for whole classes/files) so it only runs when `--run-performance` is used.
- Document why the test is performance-marked in its docstring when the reason is not obvious.


## Development Workflow

This is standard ISO prolog implementation. The parser should parse standard prolog syntax.

### Adding a New Built-in

1. **Choose a module in `vibeprolog/builtins/`** that fits (e.g., `arithmetic.py`, `list_ops.py`, `type_tests.py`).
2. **Implement a static handler** with the signature `(args, subst, engine)` and annotate return types.
3. **Register the predicate** in the module's `register(registry, engine_ref=None)` using `register_builtin`.
4. **Write tests in `tests/`** covering success and failure cases.
5. **Update documentation** (`CLAUDE.md`/`FEATURES.md`) to reflect the new predicate.
6. **Run tests**:
   ```bash
   uv run pytest tests/test_new_builtins.py -v
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nlothian/Vibe-Prolog](https://github.com/nlothian/Vibe-Prolog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
