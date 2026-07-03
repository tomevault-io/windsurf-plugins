---
trigger: always_on
description: This document provides instructions for GitHub Copilot when working on this repository.
---

# Copilot Instructions

This document provides instructions for GitHub Copilot when working on this repository.

## Code Quality Requirements

Before completing any code changes, you MUST run the following checks and ensure they pass:

### 1. Linting with Ruff

Run the linter to check for code style and potential issues:

```bash
uv run ruff check .
```

If there are issues, you can auto-fix many of them with:

```bash
uv run ruff check . --fix
```

### 2. Format Check with Ruff

Ensure code is properly formatted:

```bash
uv run ruff format --check .
```

To auto-format:

```bash
uv run ruff format .
```

### 3. Type Checking with Ty

Run the type checker to ensure type safety:

```bash
uv run ty check
```

### 4. Run Affected Tests

Rather than running the full test suite, identify and run only the tests affected by your changes. The full test suite is enforced by CI on every PR.

```bash
# Map changed source files to their test files:
#   src/copilot_session_tools/database.py  → tests/test_database.py
#   src/copilot_session_tools/scanner/     → tests/test_scanner.py
#   src/copilot_session_tools/cli.py       → tests/test_cli.py
#   src/copilot_session_tools/web/         → tests/test_webapp.py
#   src/copilot_session_tools/markdown_exporter.py → tests/test_markdown_exporter.py
#   src/copilot_session_tools/html_exporter.py     → tests/test_html_exporter.py

# Example: if you changed database.py and scanner/
uv run pytest tests/test_database.py tests/test_scanner.py -v
```

If unsure which tests are affected, run the full suite:

```bash
uv run pytest tests/ --ignore=tests/test_webapp_e2e.py -v
```

## Workflow Summary

Before committing any changes:

1. Run `uv run ruff check .` - fix any linting errors
2. Run `uv run ruff format .` - format the code
3. Run `uv run ty check` - fix any type errors  
4. Run `uv run pytest tests/test_<affected>.py` - run affected tests (CI runs the full suite)

Linting, formatting, and type checks are also enforced by a sessionEnd hook. The full test suite is enforced in CI via GitHub Actions and must pass before merging.

## Rendering Architecture

The rendering pipeline is: **raw session files → scanner → database → renderers**.

1. **Scanner** (`src/copilot_session_tools/scanner/`) parses CLI `.jsonl` and VS Code `.json`/`.jsonl` files into `ChatMessage` objects with parent/child nesting (e.g., agent tool invocations are children of the agent message).
2. **Database** stores parsed messages with relationships intact.
3. **Renderers** consume the stored data:
   - **Web viewer** (`src/copilot_session_tools/web/`, `session.html` Jinja template) — Flask app at `http://127.0.0.1:5000/`
   - **Markdown exporter** (`markdown_exporter.py`) — static `.md` export
   - **HTML exporter** (`html_exporter.py`) — static `.html` export

Key rendering rules:
- Agent/subagent blocks nest inside assistant messages as `<details class="subagent-block">`
- Tool invocations render **inline** within the message that triggered them, not in a trailing section
- Both CLI and VS Code sessions should render identically for equivalent content

## Visual Self-Verification

This project renders Copilot chat sessions as HTML (web viewer) and Markdown (export). Rendering changes are invisible to unit tests alone — you **must** verify them visually.

**IMPORTANT:** For any change to scanner, exporters, or web templates — you MUST visually verify BEFORE committing. Do not wait to be asked. This is a required step:
1. Start the web server (`uv run copilot-session-tools web`)
2. Capture Playwright screenshots of affected sessions
3. View the screenshots yourself to verify correctness
4. Present visual evidence to the user before committing

For rendering changes, **do not commit until the user has seen and approved the visual result**. Present screenshots or a Showboat demo first, then ask if they want to commit.

### Spot-checking with real sessions

When verifying rendering changes, spot-check against **real archived sessions** via the web viewer, not just test fixtures. Use sessions that actually contain the feature being modified (e.g., sessions with real subagent events for agent rendering work, not sessions that merely mention "subagent" in text).

### Snapshot baselines (automated regression detection)

Golden-file baselines in `tests/snapshots/baselines/` catch unintended rendering changes. The `tests/test_snapshot.py` suite parses real session fixtures through the scanner and exporters, then diffs the output against saved baselines using `pytest-regressions`.

```bash
# Run snapshot tests (compares against baselines)
uv run pytest tests/test_snapshot.py -v

# Regenerate only failing baselines after intentional changes
uv run pytest tests/test_snapshot.py --force-regen -v

# Regenerate ALL baselines from scratch
uv run pytest tests/test_snapshot.py --regen-all -v
```

**Important:** Baselines are gitignored but shared across worktrees via NTFS junctions pointing to `C:\_SRC\copilot-repository-tools.snapshots\`. When updating baselines for a PR, force-add them: `git add -f tests/snapshots/baselines/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arithmomaniac/copilot-session-tools](https://github.com/Arithmomaniac/copilot-session-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
