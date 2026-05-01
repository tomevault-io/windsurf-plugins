---
trigger: always_on
description: Provides SQLite database operations for financial data including:
---

# AGENTS.md - Oxidation Finance V2.0 Development Guide

**⚠️ CRITICAL: READ THIS FILE FIRST**  
This is the project's "black box" - the single source of truth for context restoration.

---
## 🧠 PROJECT MEMORY CORE

### Project Identity
- **Name**: CWZS (氧化加工财务系统)
- **Root**: `E:\python\CWZS`
- **Main Version**: v2.0 (`oxidation_finance_v20/`)
- **Active Development**: YES
- **Last Major Refactor**: 2025-02 (Service layer + security hardening)

### Known Issues (Active)

| File | Line(s) | Issue Type | Fix Status | Description |
|------|---------|------------|-------------|-------------|
| `tests/test_web_api.py` | 23 | ImportError | ⚠️ Open | `ModuleNotFoundError: No module named 'oxidation_finance_v20.web_app'` - web_app missing |
| `examples/generate_comprehensive_demo.py` | 20 | Import Error | ✅ Fixed | Incorrect `sys.path` causing `ModuleNotFoundError` (fixed to `parent.parent.parent`) |
| `tools/setup_wizard.py` | 193, 319, 335 | Bare except | ✅ Fixed | Replaced `except:` with `except sqlite3.Error:` |
| `tools/smart_calculator.py` | 338 | Bare except | ✅ Fixed | Replaced `except:` with `except Exception:` |
| `tools/quick_panel.py` | 270 | Bare except | ✅ Fixed | Replaced `except:` with `except Exception:` |
| `tools/data_quality_check.py` | 246 | Bare except | ✅ Fixed | Replaced `except:` with `except sqlite3.Error:` |
| `database/schema.py` | 240 | SQL Injection | ✅ Fixed | f-string DROP TABLE replaced with allowlist validation |
| `tools/setup_wizard.py` | 178, 288 | SQL Injection | ✅ Fixed | f-string SELECT replaced with `.format()` + allowlist |
| `tools/backup_restore.py` | 236 | SQL Injection | ✅ Fixed | f-string SELECT replaced with `.format()` + allowlist |

**Legacy Files** (not in main version):
- Redundant test runners in root: ~15 `run_*.py` and `verify_*.py` scripts (consider consolidating)
- Old version directories moved to `deprecated_versions/`

### Code Quality Status

| Category | Status | Notes |
|----------|--------|------|
| Type Safety | ✅ 95% | Decimal/UUID/Optional properly used |
| SQL Injection | ✅ Fixed | All table names validated via allowlist |
| Error Handling | ✅ Improved | Bare except clauses eliminated |
| Service Layer | ✅ Complete | `services/__init__.py` implements separation |
| Test Coverage | ✅ 70+ core tests passing | Database, order, user, accrual all green |
| Demo Generator | ✅ Fixed | `examples/generate_comprehensive_demo.py` runs successfully |
| Quick Panel | ✅ Enhanced | Friendly error message for missing database |

### Recent Commits (Top 5)
```bash
acd99de refactor(project-structure): reorganize repository and improve documentation
829888b security: fix SQL injection vulnerabilities with table name allowlists
2456c03 feat: add WebService layer for improved separation of concerns
dfa1c25 fix: critical type safety and None-check errors in FinanceManager
088d0f0 fix: resolve critical test failures and improve web UI
```

### Current State
- **Service Layer**: ✅ Implemented in `services/__init__.py`
- **SQL Injection**: ✅ Fixed (table allowlists added)
- **Type Safety**: ✅ Mostly compliant (Decimal, UUID, Optional)
- **Test Coverage**: 426 tests passing (excluding web_api import issue)
- **Web Layer**: ⚠️ `web_app.py` not using service layer (needs integration)

### AI Collaboration Protocol

**CRITICAL RULES:**
1. **Always start here** - Read this file before any work
2. **Update this file** after any significant change
3. **Document decisions** in the "Known Issues" and "Recent Context" sections

**Workflow:**
- Phase 0: Read AGENTS.md → Understand context
- Phase 1: Run `pytest` → Check current state
- Phase 2: Create todo list → Get approval (if complex)
- Phase 3: Implement → Verify tests pass
- Phase 4: Update AGENTS.md → Commit with clear message

**When committing:**
```bash
git add <changed files>
git commit -m "type(scope): brief description

Detailed explanation if needed (wrap at 72 chars)"
```

**DO NOT:**
- Delete old versions without documenting in AGENTS.md
- Merge branches without updating this file
- Skip test runs before committing

---

## Project Overview

**Oxidation Finance V2.0** is a financial management system for small oxidation processing enterprises, built with Python 3.8+. The main development directory is `oxidation_finance_v20/`.

## Build/Lint/Test Commands

### Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_database.py

# Run specific test class
pytest tests/test_database.py::TestDatabaseBasics

# Run specific test
pytest tests/test_database.py::TestDatabaseBasics::test_customer_crud

# Run with coverage
pytest --cov=oxidation_finance_v20

# Run specific marker
pytest -m unit          # Unit tests only
pytest -m integration   # Integration tests
pytest -m property      # Property-based tests
pytest -m database      # Database tests
pytest -m slow          # Slow tests (use sparingly)

# Run with hypothesis (property-based)
pytest tests/ --hypothesis-show-statistics
```

### Quick Test Scripts

```bash
# Quick test runners (located in oxidation_finance_v20/)
python quick_test.py              # Basic quick test
python run_accrual_tests.py      # Accrual tests
python run_audit_tests.py        # Audit tests
python run_backup_tests.py        # Backup/restore tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tel9980/cw](https://github.com/tel9980/cw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
