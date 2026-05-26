---
trigger: always_on
description: _Last updated: 2025-08-15_
---

# Claude Code Instructions

_Last updated: 2025-08-15_

## Purpose
This file provides context and guidelines for Claude Code when contributing to the **dji-drone-metadata-embedder** repository. It ensures consistent, high-quality contributions aligned with the project's production-ready status.

---

## 1. Repository Context

- **Repository:** [CallMarcus/dji-drone-metadata-embedder](https://github.com/CallMarcus/dji-drone-metadata-embedder)
- **Default branch:** `master`
- **Language:** Python 3.10–3.12
- **Status:** Production-ready (All M1-M4 milestones completed)
- **CI/CD:** GitHub Actions
- **Testing:** `pytest` for unit tests, `validation_tests/` for E2E tests
- **Package management:** [`uv`](https://docs.astral.sh/uv/) with `pyproject.toml` and `uv.lock`
- **CLI entry point:** `dji-embed`

---

## 2. Project Architecture

### Core Components
```
src/dji_metadata_embedder/
├── cli.py                    # Main CLI with Click commands
├── embedder.py               # Core SRT→MP4 metadata embedding
├── telemetry_converter.py    # GPX/CSV export functionality
├── metadata_check.py         # Metadata verification tool
├── dat_parser.py            # DAT flight log parsing
├── per_frame_embedder.py    # Frame-by-frame processing
├── utilities.py             # Shared utility functions
├── core/
│   ├── processor.py         # Processing pipeline
│   └── validator.py         # SRT validation & drift detection
└── utils/
    ├── dependency_manager.py # FFmpeg/ExifTool management
    └── system_info.py       # System diagnostics
```

### Key Features
- **Batch processing** of DJI drone footage (MP4 + SRT pairs)
- **GPS metadata embedding** via FFmpeg (no re-encoding)
- **Multiple DJI formats**: Mini 3/4 Pro, Air 3, Avata 2, Mavic 3 Enterprise
- **Export formats**: JSON, GPX, CSV
- **Privacy controls**: GPS redaction (drop/fuzz)
- **Cross-platform**: Windows, macOS, Linux

---

## 3. Development Guidelines

### Code Style & Standards
- **PEP 8 compliance** enforced via `ruff`
- **Type hints** for all new functions
- **Descriptive names** for variables and functions
- **Document regex patterns** with inline comments
- **No breaking changes** to existing API without major version bump

### Testing Requirements
- **All changes must have tests** in `tests/` directory
- **Use golden fixtures** in `samples/` for integration tests
- **Cross-platform validation** on Windows & Linux (Python 3.10-3.12)
- **CLI smoke tests** for new commands/options
- **Run locally before committing:** `uv run pytest -q`

### Commit Message Format
Use [Conventional Commits](https://www.conventionalcommits.org/) for automatic changelog generation:

```
feat(cli): add new validate command
fix(parser): handle malformed SRT timestamps
docs: update troubleshooting guide
ci: improve test matrix coverage
chore: remove deprecated scripts
```

**Types:**
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation only
- `ci:` - CI/CD changes
- `test:` - Test changes
- `chore:` - Maintenance tasks
- `refactor:` - Code refactoring

---

## 4. Branch Naming Convention

```
feat/issue-<NUMBER>-<short-description>     # New features
fix/issue-<NUMBER>-<short-description>      # Bug fixes
docs/<description>                           # Documentation
ci/<description>                             # CI/CD improvements
claude/<session-id>                          # Claude Code sessions
```

**Important:** Feature branches are deleted after merge to keep the repository clean. See `HOUSEKEEPING.md` for maintenance guidelines.

---

## 5. Working with GitHub Issues

When implementing GitHub issues:

1. **Read the entire issue** including acceptance criteria
2. **Create a feature branch** following naming convention above
3. **Make minimal changes** to satisfy requirements
4. **Add/update tests** and documentation
5. **Run tests locally:** `uv run pytest -q`
6. **Open a PR** with conventional commit format in title
7. **Reference the issue** in PR description: `Closes #123`

### PR Template Checklist
- [ ] Tests added/updated for new functionality
- [ ] Documentation updated (if user-facing changes)
- [ ] Sample fixtures tested (if parser changes)
- [ ] Cross-platform compatibility verified
- [ ] No breaking changes to existing functionality
- [ ] Conventional commit format used in PR title

---

## 6. Common Tasks

### Setup Development Environment
```bash
# Clone and setup
git clone https://github.com/CallMarcus/dji-drone-metadata-embedder.git
cd dji-drone-metadata-embedder

# Install uv if you don't have it
curl -LsSf https://astral.sh/uv/install.sh | sh   # macOS/Linux
# Windows: powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Install the project + dev tools into a managed .venv
uv sync --extra dev

# Verify setup
uv run pytest -q
uv run dji-embed --help
uv run dji-embed doctor
```

### Running Tests
```bash
# Unit tests (fast)
uv run pytest -q

# Specific test file
uv run pytest tests/test_parsing.py -v

# Integration tests with samples
uv run pytest tests/test_golden_fixtures.py -v

# E2E validation tests
uv run python validation_tests/run_all_tests.py
```

### Testing with Sample Fixtures
```bash
# Check sample files
uv run dji-embed check samples/

# Test embedding with Air 3 samples
uv run dji-embed embed samples/air3/ --verbose

# Validate SRT/MP4 sync

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CallMarcus/dji-drone-metadata-embedder](https://github.com/CallMarcus/dji-drone-metadata-embedder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
