---
trigger: always_on
description: This file defines coding standards, workflows, and conventions for AI assistants (Droid/Claude) working on SecureVibes. Factory automatically reads this file to ensure consistent, high-quality results.
---

# SecureVibes Agent Guidelines

This file defines coding standards, workflows, and conventions for AI assistants (Droid/Claude) working on SecureVibes. Factory automatically reads this file to ensure consistent, high-quality results.

---

## Git Workflow

### Staging and Committing

**STOP POINT: After `git add` + commit message suggestion**

When making changes, Droid should:
1. ✅ Make changes (create, edit, delete files)
2. ✅ Stage changes with `git add <files>`
3. ✅ Generate a commit message following repository style (check `git log --oneline -10`)
4. ✅ Display the full command: `git commit -m "suggested message"`
5. 🛑 **STOP HERE** - Wait for manual execution

**I will then:**
- Review staged files with `git status` (optional)
- Review changes with `git diff --cached` (optional)
- Execute or modify the suggested commit command
- Push manually when ready

**Droid should NEVER:**
- ❌ Run `git commit` (unless explicitly requested)
- ❌ Run `git push` (unless explicitly requested)
- ❌ Commit without showing what's staged first

**Example:**
```bash
# Droid does:
git add packages/core/securevibes/scanner.py tests/test_scanner.py

# Then outputs:
"✅ Files staged: scanner.py, test_scanner.py

Suggested commit command:
git commit -m "feat: Add file type filtering to scanner"

Review with 'git status' or 'git diff --cached' if needed."
```

---

## Code Style

### Python Standards

- **Follow PEP 8** for all Python code
- **Type hints**: Use type annotations for function signatures
- **Docstrings**: Use for all public functions/classes (Google style)
- **Line length**: 100 characters (configured in pyproject.toml)
- **Imports**: Group in standard order (stdlib, third-party, local)

### Formatting Tools

- **Black**: Configured in `pyproject.toml` (line-length = 100)
- **Ruff**: Linter configured in `pyproject.toml`

### Code Patterns

**Prefer:**
- ✅ Early returns over nested conditionals
- ✅ Composition over inheritance
- ✅ Explicit over implicit
- ✅ Small, focused functions (under 30 lines)
- ✅ Descriptive variable names

**Avoid:**
- ❌ Broad exception catching without re-raising
- ❌ Mutable default arguments
- ❌ Global state
- ❌ Magic numbers (use constants)

---

## Testing

### Test Requirements

**Before completing any feature:**
1. ✅ Run relevant tests: `pytest packages/core/tests/`
2. ✅ Check for import errors
3. ✅ Verify documentation examples still work
4. ✅ Ensure all tests pass

**When changing code, always update together:**
- ✅ Code implementation
- ✅ Tests for the code
- ✅ Documentation (docstrings, README, etc.)
- ✅ Remove docs for deleted features

**Never:**
- ❌ Update code without updating tests
- ❌ Delete features without removing docs
- ❌ Batch documentation updates separately

### Test Commands

```bash
# Run all tests
pytest packages/core/tests/

# Run with coverage
pytest packages/core/tests/ --cov=securevibes --cov-report=term-missing

# Run specific test file
pytest packages/core/tests/test_scanner.py

# Run with verbose output
pytest packages/core/tests/ -v
```

### Test Structure

Follow the pattern in `packages/core/tests/test_scanner.py`:
- Use descriptive test names: `test_<function>_<scenario>_<expected_result>`
- Use fixtures for common setup
- Keep tests focused and independent
- Use `pytest-asyncio` for async tests

---

## Architecture

### Multi-Agent Pattern

SecureVibes uses a specialized agent architecture:
- **Assessment Agent**: Maps codebase architecture
- **Threat Modeling Agent**: STRIDE threat analysis
- **Code Review Agent**: Security vulnerability detection
- **Report Generator**: Compiles results

**Follow existing patterns in:**
- `packages/core/securevibes/agents/definitions.py` - Agent definitions
- `packages/core/securevibes/scanner/scanner.py` - Scanner orchestration

### Package Structure

```
packages/core/securevibes/
├── agents/          # Agent definitions and prompts
├── cli/             # CLI interface
├── config.py        # Configuration management
├── models/          # Data models (Issue, Result)
├── prompts/         # Agent prompt templates
├── reporters/       # Output formatters (JSON, Markdown)
└── scanner/         # Core scanning logic
```

**When adding new features:**
- Scanners go in `scanner/`
- Reporters go in `reporters/` with interface matching existing ones
- Models go in `models/` with clear type definitions
- Agent prompts go in `prompts/` as `.txt` files

---

## Common Commands

### Development

```bash
# Run tests
pytest packages/core/tests/

# Format code
black packages/core/securevibes packages/core/tests

# Lint code
ruff check packages/core/securevibes packages/core/tests

# Install in development mode
pip install -e packages/core

# Install with dev dependencies
pip install -e "packages/core[dev]"
```

### Before Declaring "Done"

Run this verification sequence:
```bash
# 1. Format
black packages/core/securevibes packages/core/tests

# 2. Lint
ruff check packages/core/securevibes packages/core/tests

# 3. Test
pytest packages/core/tests/

# 4. Check imports
python -c "import securevibes; print('OK')"
```

---

## Documentation

### Multiple Locations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anshumanbh/securevibes](https://github.com/anshumanbh/securevibes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
