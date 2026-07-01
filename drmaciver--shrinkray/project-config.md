---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

Shrink Ray is the best test-case reducer in the world. It's a personal hobby/research project, but it's important that it be high-quality, robust, and user-friendly software so it can be widely used. C-Reduce is the only comparable tool, and Shrink Ray has surpassed it.

Because this is a hobby project worked on haphazardly, there's accumulated technical debt and the codebase quality is uneven. Don't assume existing code patterns are correct just because they exist - much of it reflects partial refactors or rushed work.

## Claude's Role and Quality Expectations

The goal is to achieve software quality beyond what a single person working part-time can accomplish. This means taking on maintenance work (fixing type errors, improving test coverage, refactoring) and doing it *well*.

### Be Meticulous, Not Sloppy

- **Fix problems properly** - Don't just suppress errors or use workarounds. If there's a type error, understand why and fix the underlying issue. If a test is hard to write, think about what would make the code more testable.
- **Don't give up on hard things** - When something seems difficult (like covering certain code paths), get creative. Refactor for testability, use Hypothesis for property-based testing, run subprocesses and gather coverage, write generators to produce edge-case inputs. Persistence matters more than speed.
- **Self-review before presenting work** - Before committing or presenting code, review it critically: "Is this sloppy? Did I take shortcuts? If I were reviewing someone else's PR with this code, what would I flag?" This catches many issues before they waste the maintainer's time.

### Avoid Suppressions

Never use these as shortcuts - they're code smells that indicate a problem to fix:

- **`# type: ignore`** - Fix the type error properly. Use proper type annotations, add type guards, or refactor to make types work. If you're adding `type: ignore`, ask yourself why the code is confusing the type checker and fix that.
- **`# pragma: no cover`** - Write a test that covers the code. If coverage is hard to achieve, refactor for testability (extract methods, use dependency injection, etc.).
- **`# pragma: no branch`** - Same as above - find a way to exercise the branch.
- **`# noqa`** - Fix the lint error. If the linter is wrong, it's usually a sign the code could be clearer.

When tempted to add a suppression, instead:
1. Understand why the tool is complaining
2. Fix the underlying issue (refactor, add types, write tests)
3. Only if the tool is genuinely wrong AND there's no cleaner solution, consider suppression - but this should be rare

**Allowed exceptions** (where suppression is acceptable):
- `# noqa: B027` for intentionally empty methods on abstract classes that serve as optional override hooks (no better alternative exists)

If you encounter a case where suppression seems genuinely necessary and principled, ask the maintainer about adding it to this list.

### Commits

- Make small, logically self-contained commits
- Each commit should ideally be lint-clean and pass tests
- Commits are a good checkpoint for self-review
- Just commit when ready - don't ask for permission
- Use the `/checkpoint` skill to ensure consistent quality at each commit
- **Always use `git add` with specific file paths** - Never use `git add -A`, `git add .`, or `git add <directory>`. Always list the specific files you intend to stage. This prevents accidentally committing unrelated files (test scripts, debug files, etc.).

### No Backward Compatibility

Shrink Ray is a standalone application, not a library. Do not add backward compatibility shims, re-exports, or compatibility layers when refactoring. When moving code between modules, update all imports directly rather than re-exporting from the old location.

### CLAUDE.md as Source of Truth

This file is the source of truth for project conventions. However:
- Update it based on feedback from the maintainer
- Update it based on your own judgment about what would improve the project
- Ask about specific style decisions you notice and record them here

## Build and Development Commands

```bash
# Install dependencies
just install

# Run tests (skips slow tests by default)
just test

# Run all tests including slow ones
just test -m ""

# Run a single test file
just test tests/test_sat.py

# Run a specific test
just test tests/test_sat.py::test_name -v

# Lint and type-check
just lint

# Run shrinkray CLI
uv run shrinkray <interestingness_test> <file_to_reduce>
```

Note: Some tests require `minisat` to be installed (`apt-get install minisat` on Ubuntu).

## Detailed Documentation

See the `notes/` directory for detailed architecture documentation:
- `notes/core-abstractions.md` - ReductionProblem, Format, passes, pumps
- `notes/patching-system.md` - How parallel patch application works
- `notes/reduction-passes.md` - Catalog of all reduction passes with examples
- `notes/parallelism-model.md` - WorkContext and speculative execution

## Architecture Overview


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DRMacIver/shrinkray](https://github.com/DRMacIver/shrinkray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
