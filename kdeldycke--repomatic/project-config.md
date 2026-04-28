---
trigger: always_on
description: This repository serves as the **canonical reference** for conventions and best practices. When Claude is used in any repository that uses the `repomatic` CLI and its [`[tool.repomatic]` configuration](readme.md#toolrepomatic-configuration), it should follow the same conventions defined here—including the structure and guidelines of this `claude.md` file itself.
---

# Development guide

## Downstream repositories

This repository serves as the **canonical reference** for conventions and best practices. When Claude is used in any repository that uses the `repomatic` CLI and its [`[tool.repomatic]` configuration](readme.md#toolrepomatic-configuration), it should follow the same conventions defined here—including the structure and guidelines of this `claude.md` file itself.

In other words, downstream repositories should mirror the patterns established here for code style, documentation, testing, and design principles.

**Contributing upstream:** If Claude spots inefficiencies, potential improvements, performance bottlenecks, missing features, or opportunities for better adaptability in the `repomatic` CLI, its configuration, the reusable workflows, or this `claude.md` file itself, it should propose these changes upstream via a pull request or issue at [`kdeldycke/repomatic`](https://github.com/kdeldycke/repomatic/issues). This benefits all downstream repositories.

**Upstream runtime dependency boundary:** Downstream repos must have only **one runtime dependency** on the upstream repository: reusable workflow `uses:` calls (e.g., `kdeldycke/repomatic/.github/workflows/autofix.yaml@vX.Y.Z`). These are version-pinned to a git tag, giving downstream repos control over when to upgrade. All other references to the upstream (documentation links in PR body templates, footer attribution) are **informational only** — they do not affect functionality if the upstream is unavailable. Do not introduce new runtime dependencies on the upstream repo (e.g., Renovate shareable presets, remote config extends, API calls to upstream) as they create unversioned coupling where an upstream breakage would cascade to all downstream repos simultaneously.

## Commands

### Testing

```shell-session
# Run all tests with coverage.
$ uv run --group test pytest

# Run a single test file.
$ uv run --group test pytest tests/test_changelog.py

# Run a specific test.
$ uv run --group test pytest tests/test_changelog.py::test_function_name

# Run tests in parallel.
$ uv run --group test pytest -n auto
```

### Type checking

```shell-session
$ uv run --group typing repomatic run mypy -- repomatic
```

### Running the CLI

```shell-session
# Run locally during development.
$ uv run repomatic --help

# Try without installation using uvx.
$ uvx -- repomatic --help
```

## Documentation requirements

### Scope of `claude.md` vs `readme.md`

- **`claude.md`**: Contributor and Claude-focused directives—code style, testing guidelines, design principles, and internal development guidance.
- **`readme.md`**: User-facing documentation for the `repomatic` CLI and `[tool.repomatic]` configuration—installation, usage, and the workflows that implement them.

When adding new content, consider whether it benefits end users (`readme.md`) or contributors/Claude working on the codebase (`claude.md`).

### Keeping `claude.md` lean

`claude.md` must contain only conventions, policies, rationale, and non-obvious rules that Claude cannot discover by reading the codebase. Actively remove:

- **Structural inventories** — project trees, module tables, workflow lists. Claude can discover these via `Glob`/`Read`.
- **Code examples that duplicate source files** — YAML snippets copied from workflows, Python patterns visible in every module. Reference the source file instead.
- **General programming knowledge** — standard Python idioms, well-known library usage, tool descriptions derivable from imports.
- **Implementation details readable from code** — what a function does, what a workflow's concurrency block looks like. Only the *rationale* for non-obvious choices belongs here.

### Changelog and readme updates

Always update documentation when making changes:

- **`changelog.md`**: Add a bullet point describing **what** changed (new features, bug fixes, behavior changes), not **why**. Keep entries concise and actionable. Justifications and rationale belong in documentation (`readme.md`, Sphinx docs) or code comments, not in the changelog.
- **`readme.md`**: Update relevant sections when adding/modifying workflow jobs, CLI commands, or configuration options.

### Documentation sync

The following documentation artifacts must stay in sync with the code. When changing any of these, update the others:

- **CLI output in `readme.md`**: The inline `uvx -- repomatic` help block, `--version` output, and development version output must match actual CLI output. Re-run the commands and update the pasted text.
- **Version references in `readme.md`**: The `--version` examples and example workflow `@vX.Y.Z` reference must reflect the latest released version.
- **Workflow job descriptions in `readme.md`**: Each `.github/workflows/*.yaml` workflow section must document all jobs by their actual job ID, with accurate descriptions of what they do, their requirements, and skip conditions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kdeldycke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
