---
trigger: always_on
description: Looking for instructions tailored to AI coding assistants?
---

# Agent Guidance

Looking for instructions tailored to AI coding assistants?

Use [`CLAUDE.md`](./CLAUDE.md). It is the canonical, maintained guide for all
agents (Claude, Cursor, ChatGPT, etc.) that interact with this repository.

## PR Title Conventions for AI Assistants

This project uses **Release Please** for automated changelog generation.
When creating PRs, use **conventional commit format** for PR titles:

### Required Format

```text
<type>[optional scope]: <description>
```

### Supported Types

- `feat:` New features (triggers MINOR version)
- `fix:` Bug fixes (triggers PATCH version)
- `revert:` Reverts of previous changes (triggers PATCH version)
- `docs:` Documentation changes (triggers PATCH version)
- `chore:` Maintenance, dependencies (triggers PATCH version)
- `refactor:` Code refactoring (triggers PATCH version)
- `style:` Formatting, linting (triggers PATCH version)
- `test:` Test changes (triggers PATCH version)
- `perf:` Performance improvements (triggers MINOR version)
- `ci:` CI/CD changes (triggers PATCH version)
- `build:` Build system changes (triggers PATCH version)

When reverting a change, use `revert:` (not the default `Revert "..."` subject)
so Release Please records it under the **Reverts** section of the changelog.
See `CONTRIBUTING.md` for the full revert workflow.

### Breaking Changes

Add `!` to indicate breaking changes (triggers MAJOR version):

- `feat!: remove deprecated API`
- `fix!: change database schema`

### Examples

```text
feat: add OAuth authentication support
fix: resolve Docker build failure
docs: update installation guide
chore: bump FastAPI dependency
feat!: remove deprecated authentication system
```

### Why This Matters

- PR titles become commit messages after squash
- Every merged PR creates/updates a Release Please PR
- Release happens when the Release Please PR is merged
- Automatic changelog generation from PR titles
- Professional release notes for users

### Release Workflow

1. Merge any PR → Release Please creates/updates a release PR
2. Review the release PR (version bump + changelog)
3. Merge the release PR when ready to release
4. Release Please publishes the release and triggers package publishing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for detailed guidelines.

## Documentation Requirements

Every new feature or significant change **must** include documentation
updates as part of the same PR, not as follow-up work. Update whichever
of the following apply:

- **Docs site** (`vibetuner-docs/docs/`) — the relevant page(s) for
  the feature area
- **`llms.txt`** (`vibetuner-docs/docs/llms.txt`) — concise feature
  entry in the Features section
- **`llms-full.txt`** (`vibetuner-docs/docs/llms-full.txt`) — detailed
  documentation with code examples
- **`vibetuner-template/AGENTS.md`** — if the feature affects
  scaffolded projects (and the corresponding `.claude/rules/` file
  if applicable)

If you are unsure which docs to update, update all four. Omitting docs
from a feature PR is a review blocker.

## Testing Scaffold Changes from a Branch

When working on scaffold-related changes (template updates, CLI improvements), you can test
the scaffolding CLI directly from a branch without publishing to PyPI:

```bash
# Test scaffold command from a specific branch
uvx git+https://github.com/alltuner/vibetuner@BRANCH_NAME#subdirectory=vibetuner-py scaffold new --help

# Create a test project using the branch
uvx git+https://github.com/alltuner/vibetuner@BRANCH_NAME#subdirectory=vibetuner-py scaffold new /tmp/test-project
```

The scaffold command also accepts a `-b` parameter to specify the branch dynamically:

```bash
# Specify branch with -b parameter (when available)
uvx git+https://github.com/alltuner/vibetuner#subdirectory=vibetuner-py scaffold new -b BRANCH_NAME /tmp/test-project
```

**When to use this:**

- Testing scaffold changes before merging a PR
- Verifying bug fixes in template or CLI
- Allowing others to test your changes before review
- CI/CD integration testing

**Note:** The `.tmp/` directory at the root is gitignored. Use it for testing scaffold
commands if you have difficulty accessing external directories like `/tmp`:

```bash
# Test scaffolding in the repo's .tmp directory
uvx git+https://github.com/alltuner/vibetuner@BRANCH_NAME#subdirectory=vibetuner-py scaffold new ./.tmp/test-project
```

See `vibetuner-docs/docs/development.md` for complete development workflows.

## Repository Justfile Commands

For developers working on the vibetuner repository itself, the root justfile provides commands for
managing the monorepo.

Run `just` to see all available commands.

### Dependencies

```bash
just update-all              # Update all package dependencies (py, js, template, root)
just update-and-commit       # Update all dependencies and commit changes
just update-py               # Update Python dependencies in vibetuner-py
just update-js               # Update JavaScript dependencies in vibetuner-js
just update-template         # Update dependencies in vibetuner-template
just update-root             # Update root scaffolding dependencies
just sync-jinja              # Refresh vibetuner-jinja/templates from vibetuner-py
```

The `sync-jinja` recipe mirrors `vibetuner-py/src/vibetuner/templates/frontend/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alltuner/vibetuner](https://github.com/alltuner/vibetuner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
