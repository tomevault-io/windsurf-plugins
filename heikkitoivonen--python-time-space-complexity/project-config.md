---
trigger: always_on
description: This file contains instructions for AI agents (like Amp, Claude, etc.) working on this repository.
---

# AI Agent Guidelines

This file contains instructions for AI agents (like Amp, Claude, etc.) working on this repository.

## Core Rules

### Pre-Commit Verification (MANDATORY)

**Before creating ANY git commit, you MUST:**

1. Run `make check` (lint + types + tests)
   ```bash
   make check
   ```
   - All ruff linting must pass
   - All pyright type checks must pass (0 errors)
   - All pytest tests must pass (10/10)

2. Verify output shows:
   ```
   All checks passed!
   0 errors, 0 warnings, 0 informations
   ====== 10 passed ======
   ```

3. Do NOT commit if checks fail

### Example Pre-Commit Workflow

```bash
# Make changes
# ...

# Run quality checks
make lint      # Verify: All checks passed!
make format    # Fix any formatting issues
make test      # Verify: 6 passed

# Only then commit
git add .
git commit -m "Your message"
```

## Standard Commands

### Development
- `make dev` - Install with dev dependencies
- `make serve` - Start local server (http://localhost:8000)
- `make build` - Build static site

### Quality Assurance
- `make lint` - Check code quality (must pass before commit)
- `make format` - Format code automatically
- `make types` - Run pyright type checker (must pass before commit)
- `make check` - Run lint + types + tests (must pass before commit)
- `make test` - Run pytest (must pass before commit)

### Maintenance
- `make clean` - Remove build artifacts
- `make update` - Update dependencies
- `make help` - Show all available commands

## Project Structure

### Key Directories
- `/docs` - Documentation markdown files (site content)
- `/tests` - Python test files
- `/scripts` - Utility scripts (templates)
- `/data` - JSON data files for documentation

### Key Files
- `pyproject.toml` - Project config, dependencies, tool settings
- `Makefile` - Development commands
- `mkdocs.yml` - Documentation site config
- `.python-version` - Python 3.11 specification
- `uv.lock` - Dependency lock file (reproducible builds)

## Code Style & Standards

### Python Code
- Line length: 100 characters (enforced by ruff)
- No unused imports (enforced by ruff)
- Format with `make format` before commit
- All checks must pass: `make lint`

### Documentation (Markdown)
- Complexity tables required (Time, Space, Notes columns)
- Include code examples for operations
- Link to related operations
- Use admonitions for important notes:
  ```markdown
  !!! warning "Title"
      Content here
  ```

### Commit Messages
- Use imperative mood: "Add", "Fix", "Update", not "Added", "Fixed"
- Format: `Type: Brief description`
- Types: Add, Fix, Update, Refactor, Docs, Test, Chore
- Example: `Add: List complexity documentation`
- **AI Agents MUST add Co-Authored-By trailer to identify the agent:**
  ```
  Add: List complexity documentation

  Co-Authored-By: Amp <amp@ampcode.com>
  ```
  Replace "Amp <amp@ampcode.com>" with the actual AI agent name (Claude,
  Copilot, etc.) and email.

## Testing Requirements

### Before Every Commit
1. Run `make lint` - all linting must pass
2. Run `make types` - all type checks must pass
3. Run `make test` - all tests must pass
4. Never commit broken code

### Test Files Location
- `tests/test_documentation.py` - Main test file
- Must test documentation structure, data files, project files

### Running Tests
```bash
make lint           # Run linting
make types          # Run type checks
make test           # Run all tests
make check          # Run lint + types + tests (recommended before commit)
```

## Dependency Management

### Adding Dependencies
```bash
# Add production dependency
uv add package-name

# Add dev dependency
uv add --dev package-name
```

### DO NOT
- Manually edit pyproject.toml dependency lists
- Manual requirements.txt edits (uv manages this)
- Forget to commit uv.lock after adding dependencies

## Common Workflows

### Adding Documentation
1. Create markdown file in `/docs`
2. Update navigation in `mkdocs.yml`
3. Run `make audit` to update `DOCUMENTATION_STATUS.md`
4. Run `make serve` to preview
5. Run `make check` to verify
6. Commit with message: `Add: Topic name documentation`

**CRITICAL:** Always update `DOCUMENTATION_STATUS.md` after adding new documentation files:
- Run `python scripts/audit_documentation.py` to regenerate the coverage report
- Update the coverage percentages and totals in DOCUMENTATION_STATUS.md
- Move documented items from "Missing" to "Documented" sections
- Never commit without updating this file - it tracks project coverage goals

### Fixing Issues
1. Identify the problem
2. Make minimal changes
3. Run `make check` to verify
4. Commit with message: `Fix: Description of fix`

### Code Cleanup
1. Run `make format` (auto-fixes issues)
2. Verify no new issues with `make lint`
3. Run `make test` to ensure nothing broke
4. Commit with message: `Refactor: Description`

## What NOT to Do

### ❌ DO NOT
- Commit without running `make check`
- Commit if `make lint` fails
- Commit if `make types` fails
- Commit if `make test` fails
- Ignore test failures
- Manually edit lock files (use uv commands)
- Create unnecessary files in root directory
- Break existing tests without fixing them

### ✓ DO
- Always run quality checks before committing
- Keep commits focused and minimal

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heikkitoivonen/python-time-space-complexity](https://github.com/heikkitoivonen/python-time-space-complexity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
