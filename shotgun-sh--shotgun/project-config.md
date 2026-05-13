---
trigger: always_on
description: To understand how to run the evals read evals/README.md
---

# Claude Code Instructions for Shotgun

## Evals

To understand how to run the evals read evals/README.md

### Writing Eval Test Cases

When adding new eval test cases with judge rubrics:

1. **Default to LLM as a judge** - Unless the behavior being tested is purely deterministic (e.g., specific tool was called, specific field was populated), use the `expected_response` field to provide a rubric for the LLM judge to evaluate against.

2. **Write rubrics that describe correct vs incorrect behavior** - The `expected_response` field should explain:
   - What the correct behavior looks like
   - What incorrect behaviors to watch for
   - Why the distinction matters

3. **Example rubric format:**
   ```python
   expected_response="""The Router should immediately use file_requests to load the PDF file.
   Correct behavior: Set file_requests with the PDF path, provide a brief acknowledgment.
   Incorrect behavior: Asking clarifying questions, claiming inability to access files, or delegating to another agent."""
   ```

4. **Use deterministic evaluators for structural checks** - Things like `disallowed_tools`, `disallowed_delegations`, and `response_not_contains` are better as deterministic checks since they have clear pass/fail criteria.

## Architecture Documentation

For detailed architecture documentation, see:

- [Ollama/Local Model Support](docs/architecture/ollama-local-models.md) - How Shotgun integrates with local LLMs via Ollama

## Commit Message Convention

This project enforces **Conventional Commits** specification. All commit messages MUST follow this format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Required Commit Types

**IMPORTANT**: These types must stay in sync between `cz_conventional_commits` (pyproject.toml) and GitHub Actions (pr.yml).

Use these types for your commit messages:

- **feat**: A new feature
- **fix**: A bug fix  
- **docs**: Documentation only changes
- **style**: Changes that don't affect code meaning (formatting, missing semicolons, etc.)
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **perf**: Performance improvements
- **test**: Adding missing tests or correcting existing tests
- **build**: Changes that affect the build system or external dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files
- **revert**: Reverts a previous commit

### Scopes and Breaking Changes

- Add optional scope in parentheses: `feat(auth): implement OAuth2`
- For breaking changes, add `!` after the type/scope: `feat!: remove deprecated API endpoints`

## Pull Request Title Convention

PR titles MUST also follow the Conventional Commits format. When using "Squash and merge", GitHub will use the PR title as the commit message.

### Tests

- Tests must use pytest and must be seperate functions, not a pytest class.

## Testing Policy

### Test Structure

The project has two types of tests:

1. **Unit Tests** (`test/unit/`): Fast, isolated tests with no external dependencies
2. **Integration Tests** (`test/integration/`): Tests that verify complete workflows with real databases

**For Contributors:**
- All tests run in CI/CD for every PR
- Both unit and integration tests must pass
- No API keys required for tests - all external services are mocked

### Testing the TUI with Playwright MCP

When you have access to the Playwright MCP server, you can interactively test the Shotgun TUI. See `@./docs/testing-tui-with-playwright.md` for the full guide.

**Quick Start:**

1. Start the TUI web server in the background:
   ```bash
   uv run shotgun --web --port 8765 --no-update-check
   ```

2. Navigate to it with Playwright:
   ```
   browser_navigate: http://localhost:8765
   browser_wait_for: time=3
   browser_take_screenshot: filename=tui.png
   ```

**Key Techniques:**

- **Always use screenshots** - Textual renders to a canvas, so `browser_snapshot` won't show the content
- **Use keyboard navigation** - Tab/Enter to navigate, `/` for command palette, `Shift+Tab` for mode toggle
- **Type via run_code** - Standard `browser_type` doesn't work on xterm canvas:
  ```javascript
  browser_run_code: async (page) => {
    await page.click('.xterm-screen');
    await page.keyboard.type('Your message', { delay: 50 });
  }
  ```

**Common keyboard shortcuts:**
- `Tab` - Move focus
- `Enter` - Select
- `/` - Open command palette
- `Shift+Tab` - Toggle mode
- `Escape` - Close modal

## Commands for Development

- **Install dependencies**: `uv sync --all-extras`
- **Run linting**: `uv run ruff check .`
- **Run formatting**: `uv run ruff format .`
- **Run type checking**: `uv run mypy src/`
- **Create conventional commit**: `uv run cz commit`
- **Validate commit message**: `uv run cz check --commit-msg-file .git/COMMIT_EDITMSG`
- **Count tokens in folder**: `uv run python scripts/count_tokens.py .shotgun/`

### Token Counting Script

The `scripts/count_tokens.py` script counts tokens using tiktoken (cl100k_base encoding).

```bash
# Count tokens in .shotgun folder (default)
uv run python scripts/count_tokens.py

# Count tokens in any folder
uv run python scripts/count_tokens.py src/shotgun/prompts/
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shotgun-sh/shotgun](https://github.com/shotgun-sh/shotgun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
