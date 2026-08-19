---
trigger: always_on
description: This document provides context to understand the Deep Agents Python project and assist with development.
---

# Global development guidelines for the Deep Agents monorepo

This document provides context to understand the Deep Agents Python project and assist with development.

For environment setup, pre-commit installation, and the standard edit-test-lint loop, see [`libs/DEVELOPMENT.md`](libs/DEVELOPMENT.md). The rest of this document covers conventions and architecture reference.

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

#### Environment and dependency management

Use `uv` for all environment and dependency operations in this monorepo. Do not invoke `pip`, `poetry`, or `conda` directly.

- Let `uv` manage the interpreter and virtual environments — `uv sync` and `uv run` operate without manual `source .venv/bin/activate`. Do not create ad-hoc virtual environments outside the package directory.
- Each package targets its own supported Python range via its `pyproject.toml`; do not pin a global Python version. If you need an interpreter explicitly, defer to the package's `requires-python` rather than assuming system Python.
- Install dependencies explicitly through `uv sync` (optionally `--group <name>` / `--all-groups`); never let them install implicitly.
- Don't mix environments within a session, and don't add new dependencies unless strictly required — when you do, justify them (recent releases/commits, adoption).

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
- Do not include Linear issue-closing markers such as `[closes DCD-52]` in PR titles. Put issue references and closing metadata in the PR description instead.
- For version-branch sync PRs, use a title like `chore(repo): sync main into vX.Y`. Do not use `release` as the scope; PR title lint reserves `release` for the type and disallows it as a scope.
- **One PR = one releasable component.** release-please scopes commits by **changed file path**, not title scope. A bump-worthy title (`feat`/`fix`/etc.) that touches files under multiple managed packages opens a **separate release PR per package**. Keep the user-facing change in a single package-scoped PR; ship cross-package dependency / lockfile churn as a separate `chore(deps):` PR (`chore` is hidden and does not cut releases). See [Multi-component fan-out](.github/RELEASING.md#multi-component-fan-out) and [Lockfile churn fan-out](.github/RELEASING.md#lockfile-churn-fan-out).

Examples:

```txt
feat(sdk): add new chat completion feature
fix(cli): resolve type hinting issue
chore(evals): update infrastructure dependencies
test(cli): missing unit tests for `_git`
feat(cli): `--startup-cmd` flag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zishuQ/caro](https://github.com/zishuQ/caro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
