---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication

When writing, especially PRs and commit messages:
- Avoid emojis
- Avoid "LLM tells", for example:
 - Don't use bullet items with `**Heading** - description`, unless it's _absolutely required for emphasis_
 - Avoid cliches
- Be terse but informative

## Build/Test/Lint Commands

- Setup environment: `make ce`
- Run all tests: `make test`
- Run a single test: `uv run pytest bin/test/path_to_test.py::TestClass::test_method -v`
- Run static checks: `make static-checks`
- Check code style/linting: `make pre-commit`
- Install pre-commit hooks: `make install-pre-commit`
- Build lambda package: `make lambda-package`
- Build events lambda package: `make events-lambda-package`
- **NEVER USE THE SYSTEM PYTHON** - always use `uv` to invoke python or pytest or to run experiments with python syntax

## Documentation

This repository has extensive documentation in the `docs/` directory. Before making changes to a subsystem, read the relevant documentation first. Existing CLI commands, library configuration, and infrastructure patterns are already documented -- don't reinvent or guess at behavior that's written down.

## Important Workflow Requirements

- ALWAYS run pre-commit hooks before committing: `make pre-commit`
- The hooks will run tests and lint checks, and will fail the commit if there are any issues. You will need to `git add` those changed files
- Failing to run pre-commit hooks may result in style issues and commit failures
- For comprehensive validation, run `make static-checks` before committing (includes all linting and type checking)
- If static checks fail, fix the issues before committing to avoid CI failures
- **Critical**: After fixing any issues, run `make static-checks` AGAIN. Repeat until it passes completely. Only commit when `make static-checks` runs with zero errors.
- If a python lint fails **DO NOT DISABLE IT**. The lint rules are there for a reason. You **MUST NOT** override them with `noqa` or similar. If you have no choice **YOU MUST GET EXPLICIT APPROVAL FROM THE USER**.

### Correct Commit Workflow

1. Make changes
2. Run `make static-checks`
3. If it fails, fix the issues
4. Run `make static-checks` again (fixes might introduce new issues or be auto-formatted)
5. Repeat steps 3-4 until `make static-checks` passes completely
6. Only then create the commit

### Git Best Practices

- Do not use `git push --force` or `git commit --amend` unless explicitly asked
- Make regular commits for fixes rather than rewriting history

## Code Style Guidelines

- Python formatting: Black with 120 char line length
- Use type hints for Python code (mypy for validation)
  - Use modern Python 3.9+ typing syntax: `list[str]`, `dict[str, Any]`, `str | None` instead of `Optional[str]`
  - Only import `Any` from `typing` module when needed; use built-in types otherwise
  - Union types: use `X | Y` syntax instead of `Union[X, Y]`
- Follow shell best practices (shellcheck enforced)
- No unused imports or variables (autoflake enforced)
- Error handling: Use appropriate error classes and logging
- Write unit tests for new functionality (required for all new code)
- Design code to be testable: prefer pure functions and clear interfaces
- Documentation: Comments should explain "why", not "what" unless non-obvious
- Don't add comments above self-documenting code
- Maintain backwards compatibility with existing scripts
- For AWS resources, follow terraform best practices
- Never call functions within f-strings: create a variable first, then use it in the f-string
- Never use nested functions - always use class methods or module-level functions instead
- **When making a change based on feedback, search for similar patterns in the codebase** - if someone suggests a change that makes sense, check for other locations where the same improvement should be applied

## Terraform Guidelines

- When changing terraform .tf files, always run `terraform fmt` and `terraform validate` before committing (from the terraform directory)

## Testing Guidelines

- Tests are in `bin/test` and `lambda` directories with `_test.py` suffix
- Run tests with `make test` or `uv run pytest path/to/test.py`
- Test both success and error cases
- Mock external dependencies when appropriate
- **Always write tests for new functionality** - prefer testable code design
- Use pytest framework with descriptive test function names (e.g., `test_function_name_scenario`)
- Test files should import from `lib` modules directly
- Use `pytest.raises()` for exception testing with message matching
- Include both happy path and edge case scenarios
- Follow existing test patterns: simple functions, clear assertions, good docstrings

## Infrastructure Notes

This repository contains scripts and infrastructure configurations for Compiler Explorer.
Files in `/opt/compiler-explorer` are the target installation location.

## SQS Message Overflow to S3

The CE Router system supports automatic overflow of large compilation requests to S3 when they exceed SQS message size limits.

### Configuration

- **S3 Bucket**: `temp-storage.godbolt.org` (shared across all environments)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compiler-explorer/infra](https://github.com/compiler-explorer/infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
