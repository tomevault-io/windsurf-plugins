---
trigger: always_on
description: This document provides context to understand the Deep Agents Python project and assist with development.
---

# Global development guidelines for the Deep Agents monorepo

This document provides context to understand the Deep Agents Python project and assist with development.

## Project architecture and context

### Monorepo structure

This is a Python monorepo with multiple independently versioned packages:

```txt
deepagents/
├── libs/
│   ├── deepagents/  # SDK
│   ├── cli/         # CLI tool
│   ├── acp/         # Agent Context Protocol support
│   ├── evals/       # Evaluation suite and Harbor integration
│   └── partners/    # Integration packages
│       └── daytona/
│       └── ...
├── .github/         # CI/CD workflows and templates
└── README.md        # Information about Deep Agents
```

### Development tools & commands

- `uv` – Package installer and resolver (replaces pip/poetry)
- `make` – Task runner. Look at the `Makefile` for available commands and usage patterns.
- `ruff` – Linter and formatter
- `ty` – Static type checking

Local development uses editable installs: `[tool.uv.sources]`

```bash
# Run unit tests (no network)
make test

# Run specific test file
uv run --group test pytest tests/unit_tests/test_specific.py
```

```bash
# Lint code
make lint

# Format code
make format
```

#### Suppressing ruff lint rules

Prefer inline `# noqa: RULE` over `[tool.ruff.lint.per-file-ignores]` for individual exceptions. `per-file-ignores` silences a rule for the *entire* file — If you add it for one violation, all future violations of that rule in the same file are silently ignored. Inline `# noqa` is precise to the line, self-documenting, and keeps the safety net intact for the rest of the file. Add comments to justify silencing. If you can't make a good justification for the ignore, it is probably code smell and should be re-evaluated.

Reserve `per-file-ignores` for **categorical policy** that applies to a whole class of files (e.g., `"tests/**" = ["D1", "S101"]` — tests don't need docstrings, `assert` is expected). These are not exceptions; they are different rules for a different context.

```toml
# GOOD – categorical policy in pyproject.toml
[tool.ruff.lint.per-file-ignores]
"tests/**" = ["D1", "S101"]

# BAD – single-line exception buried in pyproject.toml
"deepagents_cli/agent.py" = ["PLR2004"]
```

```python
# GOOD – precise, self-documenting inline suppression
timeout = 30  # noqa: PLR2004  # default HTTP timeout, not arbitrary
```

#### PR and commit titles

Follow Conventional Commits. See `.github/workflows/pr_lint.yml` for allowed types and scopes. All titles must include a scope with no exceptions.

- Start the text after `type(scope):` with a lowercase letter, unless the first word is a proper noun (e.g. `Azure`, `GitHub`, `OpenAI`) or a named entity (class, function, method, parameter, or variable name).
- Wrap named entities in backticks so they render as code. Proper nouns are left unadorned.
- Keep titles short and descriptive — save detail for the body.

Examples:

```txt
feat(sdk): add new chat completion feature
fix(cli): resolve type hinting issue
chore(evals): update infrastructure dependencies
test(cli): missing unit tests for `_git`
feat(cli): `--startup-cmd` flag
style(cli): strip trailing annotations from `ask_user` questions
```

See [PR labeling and linting](#pr-labeling-and-linting) for more info.

#### PR descriptions

The description *is* the summary — do not add a `# Summary` header.

- When the PR closes an issue, lead with the closing keyword on its own line at the very top, followed by a horizontal rule and then the body:

  ```txt
  Closes #123

  ---

  <rest of description>
  ```

  Only `Closes`, `Fixes`, and `Resolves` auto-close the referenced issue on merge. `Related:` or similar labels are informational and do not close anything.

- Explain the *why*: the motivation and why this solution is the right one. Limit prose.
- Write for readers who may be unfamiliar with this area of the codebase. Avoid insider shorthand and prefer language that is friendly to public viewers — this aids interpretability.
- Do **not** cite line numbers; they go stale as soon as the file changes.
- Rarely include full file paths or filenames. Reference the affected symbol, class, or subsystem by name instead.
- Wrap class, function, method, parameter, and variable names in backticks.
- Skip dedicated "Test plan" or "Testing" sections in most cases. Mention tests only when coverage is non-obvious, risky, or otherwise notable.
- Call out areas of the change that require careful review.

## Core development principles

### Maintain stable public interfaces

CRITICAL: Always attempt to preserve function signatures, argument positions, and names for exported/public methods. Do not make breaking changes.

You should warn the developer for any function signature changes, regardless of whether they look breaking or not.

**Before making ANY changes to public APIs:**

- Check if the function/class is exported in `__init__.py`
- Look for existing usage patterns in tests and examples
- Use keyword-only arguments for new parameters: `*, new_param: str = "default"`
- Mark experimental features clearly with docstring warnings (using MkDocs Material admonitions, like `!!! warning`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
