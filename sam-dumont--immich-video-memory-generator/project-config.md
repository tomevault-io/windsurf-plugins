---
trigger: always_on
description: - **NEVER use `git add -f` or `--force` to bypass `.gitignore`.** If a file is ignored, it is ignored for a reason. No exceptions.
---

# Claude Code Project Instructions

## Git Safety

- **NEVER use `git add -f` or `--force` to bypass `.gitignore`.** If a file is ignored, it is ignored for a reason. No exceptions.
- Do not commit to `main` directly — use feature branches.

## Quick Start

1. **Install dev dependencies first**: `make dev`
2. Read `ARCHITECTURE.md` — it maps the full codebase structure, key classes,
   data flow, and composition architecture. This avoids needing to explore the repo.

> **Important**: Always run `make dev` before running any other make target.
> This installs all dev tools (pytest, ruff, mypy, etc.) into the project venv.
> Without it, quality checks will fail with import errors.

## Commands

All commands are available via the **Makefile**. **Always use `make` targets** —
they match what CI runs, so local results are consistent with CI. Never run raw
`ruff`, `pytest`, `mypy`, `xenon`, etc. directly — use the Makefile.

```bash
# Install dev dependencies
make dev

# Run tests
make test

# Run only what the torch-family extras unlock (face/audio-ml/demucs)
make test-extras

# Lint (ruff check)
make lint

# Auto-fix lint issues
make lint-fix

# Format code
make format

# Format check (no changes)
make format-check

# Type check (mypy)
make typecheck

# File length gate (≤800 lines per .py file)
make file-length

# Complexity gate (Xenon grade C)
make complexity

# Dead code detection (Vulture)
make dead-code

# Security lint (Bandit)
make security-lint

# Cognitive complexity gate (complexipy ≤15)
make cognitive-complexity

# Code duplication detection (jscpd, requires npm)
make duplication

# Modernization lint (refurb)
make refurb

# Dependency hygiene (deptry: hallucinated/unused/transitive deps)
make dep-check

# Regenerate the CLI reference from the Click tree, and fail on drift
make docs-cli
make docs-cli-check

# Config reference key parity with the pydantic schema
make docs-config-check


# Architectural boundary enforcement (import-linter)
make arch-check

# Diff coverage for PRs (≥80% on changed lines)
make diff-cover

# Commit message lint (conventional commits)
make commitlint

# Dependency vulnerability audit
make pip-audit

# Run ALL checks (lint + format + typecheck + file-length + complexity + test)
make check

# Full CI pipeline (all checks + advanced quality gates)
make ci

# Pre-commit hooks (runs all local hooks: lint, format, mypy, gitleaks, commitizen, file-length, complexity, dead-code, security-lint)
make pre-commit

# Self-critique check for AI code smells
make critique
```

## Rules

### Code Quality Gates (enforced in CI)

- **Lint**: ruff check must pass (`make lint`)
- **Format**: ruff format must pass (`make format-check`)
- **Type check**: mypy must pass (`make typecheck`)
- **Max file length**: 800 lines per `.py` file (`make file-length`)
- **Cyclomatic complexity**: Xenon grade C — ≤20 per function (`make complexity`)
- **Cognitive complexity**: complexipy ≤15 per function (`make cognitive-complexity`)
- **Dead code**: Vulture must pass (`make dead-code`)
- **Security**: Bandit must pass with no HIGH findings (`make security-lint`)
- **Security (cross-file)**: Semgrep must pass with no ERROR findings (`make semgrep`)
- **Modernization**: refurb must pass (`make refurb`)
- **Dependency hygiene**: deptry must pass (`make dep-check`)
- **CLI reference drift**: the generated page must match the Click tree (`make docs-cli-check`)
- **Config reference drift**: the config reference must list exactly the schema's keys (`make docs-config-check`)
- **Tests**: all tests must pass (`make test`)
- **Commit messages**: must follow [Conventional Commits](https://www.conventionalcommits.org/) (`make commitlint`)
  - Format: `type(scope): description` — e.g., `fix(api): handle timeout errors`
  - Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

### Test-Driven Development (TDD)

All new features and bug fixes **must** use TDD with vertical slices:

1. **ONE test → ONE implementation → repeat** (never write all tests first)
2. **RED**: Write one failing test for the next behavior
3. **GREEN**: Write minimal code to make it pass
4. **REFACTOR**: Clean up only after GREEN, never while RED
5. Run `make test` after each cycle to confirm

Tests must verify **behavior through public interfaces**, not implementation
details. A test should survive an internal refactor without breaking. See
`.agents/skills/tdd/SKILL.md` for the full TDD skill reference.

### Before Every Commit

Run `make ci` — this runs all CI-equivalent checks locally. If it passes
locally, CI will pass too. Use conventional commit message format (see above).

### Splitting Large Files

- Do not add new files over 800 lines — split proactively
- A PR that pushes an existing file past 800 lines OWES the split: in that PR
  or an immediately-queued follow-up. Surfing toward the 1000 hard cap is not
  an option — moving helpers elsewhere to stay at 99x is the smell, not the fix
- Split along **cohesion boundaries**, not arbitrary line counts
- Extract a service class with a Protocol contract for its dependencies
- The new module should be independently testable and importable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sam-dumont/immich-video-memory-generator](https://github.com/sam-dumont/immich-video-memory-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
