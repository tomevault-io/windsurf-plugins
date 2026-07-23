---
trigger: always_on
description: See also [the contributor documentation](docs/contributing.rst) for development workflows.
---

# CLAUDE.md

## Development Commands

See also [the contributor documentation](docs/contributing.rst) for development workflows.

### Setup

```bash
# Install dependencies (uv includes the dev group by default)
uv sync

# Install with composite support (required for rendering/compositing)
uv sync --extra composite

# Install with all groups (docs) and composite extra
uv sync --all-groups --extra composite
```

**Note**: The `composite` extra includes `aggdraw`, `scipy`, and `scikit-image` dependencies
required for layer compositing (rendering). These are optional since they may not be available
on all platforms (notably Python 3.14 on Windows).

### Building

```bash
# Build Cython modules and a wheel
uv build --wheel
```

### Testing

```bash
# Run all tests with coverage
uv run pytest

# Run specific test
uv run pytest tests/psd_tools/api/test_layers.py::test_layer_name

# Run tests without coverage
uv run pytest --no-cov
```

### Linting and Type Checking

```bash
# Run ruff linter
uv run ruff check

# Format with ruff
uv run ruff format

# Run mypy type checker
uv run mypy
```

### Documentation

```bash
# Build HTML documentation
uv run --group docs make -C docs html
# Output in docs/_build/html/
```

## Available Skills

These slash commands are available in Claude Code. Invoke them by typing `/<name>` in the chat.

**Project-defined** (`.claude/skills/`):

| Skill                         | When to use                                                                       |
| ----------------------------- | --------------------------------------------------------------------------------- |
| `/release [version]`          | Cut a new version: updates changelog, bumps version, opens a release PR.          |
| `/investigate-issue <number>` | Investigate a GitHub issue, trace the root cause, and post findings as a comment. |

**Built-in Claude Code commands**:

| Skill                  | When to use                                                                                                      |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `/code-review [--fix]` | Review the current diff for bugs and simplification opportunities. Pass `--fix` to apply findings automatically. |
| `/review`              | Review the open PR on the current branch — reads PR description, diff, and CI state.                             |
| `/security-review`     | Run a focused security review of pending changes on the current branch.                                          |

## GitHub Workflow

### Common `gh` CLI patterns

```bash
# View an issue with all comments
gh issue view 123 --repo psd-tools/psd-tools --comments

# Post a comment (use --body-file for multiline Markdown to avoid shell escaping issues)
gh issue comment 123 --repo psd-tools/psd-tools --body-file "${TMPDIR:-/tmp}/comment.md"

# View a PR (current branch)
gh pr view

# Check CI status for the current PR
gh pr checks

# List open PRs
gh pr list

# Parse JSON output with jq (more efficient than python3 for simple field extraction)
gh issue view 123 --repo psd-tools/psd-tools --json title,body,comments | jq '.comments[].body'

# Mark a PR review comment thread as resolved (GraphQL)
gh api graphql -f query='mutation { resolveReviewThread(input: {threadId: "..."}) { thread { isResolved } } }'
```

### Typical fix workflow

1. `gh issue view <N> --comments` — read the issue and comments, understand the bug
1. `git checkout -b fix/short-description` — create the branch before editing any files
1. Investigate code, write the fix, run `uv run pytest`
1. Commit and push → `gh pr create`
1. Address PR review comments → push follow-up commits
1. After merge: `git checkout main && git pull && git branch -d fix/short-description`

### Branch naming conventions

- `fix/<short-description>` — bug fixes
- `feature/<short-description>` — new features
- `release/vX.Y.Z` — release PRs (required by auto-tag workflow)
- `chore/<short-description>` — tooling, deps, CI

## Release Workflow

1. **Decide the version number** following [PEP 440](https://peps.python.org/pep-0440/) based on the changes since the last release. The auto-tag workflow recognises these forms: `v1.2.3` (release), `v1.2.3a1` / `v1.2.3b1` / `v1.2.3rc1` (pre-releases), `v1.2.3.post1` (post-release).

1. **Update the changelog**: Review `git log` since the last tag and summarize changes in `docs/changelog.rst` under the new version heading.

1. **Create a release PR**: Commit the changelog update (and any version bumps) on a branch named exactly `release/vX.Y.Z` and open a PR against `main`. Merge it once approved. The branch name is how the auto-tag workflow identifies the version.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psd-tools/psd-tools](https://github.com/psd-tools/psd-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
