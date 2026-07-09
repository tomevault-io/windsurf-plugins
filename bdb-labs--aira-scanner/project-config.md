---
trigger: always_on
description: **AIRA Scanner** (AI-Induced Risk Audit) is a research tool for detecting fail-soft patterns in software systems — especially those developed with significant AI assistance.
---

# CLAUDE.md — AIRA Scanner Development Handbook

## Project Overview

**AIRA Scanner** (AI-Induced Risk Audit) is a research tool for detecting fail-soft patterns in software systems — especially those developed with significant AI assistance.

**Core Question**: "Does the system tell the truth when it fails?"

AIRA identifies systems that:
- Return success despite incomplete or failed operations
- Degrade silently instead of failing explicitly
- Obscure true system state under error conditions
- Preserve appearance of function while weakening guarantees

**Key Facts:**
- Command-line linter + rule engine
- Detects 15 fail-soft pattern categories (C01–C15)
- Works on Python, JavaScript, and TypeScript codebases
- Published via Homebrew (`brew install BDB-Labs/aira-scanner/aira`); source installs via `pip install ./CLI`
- Research-driven: empirical validation in progress

## Repository Structure

```
aira-scanner/
├── CLI/                              # Scanner CLI + package
│   ├── aira/                         # Main package
│   │   ├── checkers/                 # Detection rule implementations
│   │   │   ├── python_checker.py     # Python AST-backed static checks
│   │   │   ├── js_checker.py         # JavaScript/TypeScript checks
│   │   │   └── test_coverage_checker.py  # Test coverage asymmetry (C14)
│   │   ├── scanner.py                # Core scanning engine (AIRAScanner)
│   │   ├── cli.py                    # CLI entry point + arg parsing
│   │   ├── __main__.py               # Entry point → cli.py:main()
│   │   ├── research.py               # Research submission (Supabase/Airtable/JSONL)
│   │   ├── llm.py                    # LLM provider routing
│   │   ├── collector.py              # Public repo collection
│   │   ├── deterministic_scan.py     # Server-side inline scan helpers
│   │   └── __init__.py               # Package init (version)
│   └── tests/                        # Test suite
│       ├── test_scanner_modes.py     # Scanner mode + edge-case tests
│       ├── test_research.py          # Research submission tests
│       ├── test_cli_scan_validation.py  # CLI validation tests
│       ├── test_cli_failure_behavior.py # CLI failure mode tests
│       ├── test_llm_routing.py       # LLM provider routing tests
│       ├── test_deterministic_scan.py   # Deterministic scan tests
│       ├── test_collector.py         # Collection tests
│       └── fixture_violations.py     # Test fixtures
├── api/                              # Web API (Vercel serverless)
├── docs/                             # Documentation (web-facing)
├── Formula/                          # Homebrew formula
├── lib/                              # Shared libraries
├── scripts/                          # Build + CI scripts
├── pyproject.toml                    # Project configuration
├── ROADMAP.md                        # Feature roadmap
├── CONTRIBUTING.md                   # Contribution guidelines
├── AIRTABLE_SCHEMA.md                # Legacy Airtable findings database
├── CHANGELOG.md                      # Version history
├── README.md                         # Quick start
├── LICENSE                           # MIT
└── .gitignore
```

## Fail-Soft Pattern Categories (C01–C15)

### C01 — Success Integrity Violations
System returns success code despite operation failure.

**Example**:
```python
try:
    db.insert(record)
except Exception:
    return True  # ✗ Returns success after failure
```

**AIRA Detection**: Flags `return True` / success-shaped dicts inside exception handlers.

### C02 — Audit & Evidence Integrity Gaps
System fails to record evidence of decision or operation outcome.

**Example**:
```python
try:
    audit_write(event)
except Exception:
    pass  # ✗ Evidence silently lost
```

**AIRA Detection**: Flags audit/write calls in try blocks with non-re-raising handlers.

### C03 — Broad Exception Suppression
Catch-all `except:` or `except Exception:` with no propagation.

**Example**:
```python
try:
    important_operation()
except:
    pass  # ✗ Silent failure
```

**AIRA Detection**: Flags bare excepts and broad Exception handlers that don't re-raise.

### C04 — Fallback & Degradation
System silently falls back to unsafe defaults instead of failing.

**AIRA Detection**: Flags `fallback`, `degraded`, `best_effort`, `fail_open` identifiers.

### C05 — Bypass & Override Paths
Escape hatches that weaken guarantees (testing, debug, force flags).

**AIRA Detection**: Flags `skip_validation`, `bypass_governance`, `force_pass`, etc.

### C06 — Ambiguous Return Contracts
Function return types don't distinguish success/failure cases.

**AIRA Detection**: Flags functions with ≥2 `return None` in different contexts.

### C07 — Parallel Logic Drift
Same operation implemented differently in different code paths. **Human review only.**

### C08 — Unsupervised Background Tasks
Async/background work with no monitoring or result collection.

**AIRA Detection**: Flags `create_task()`, `ensure_future()` fire-and-forget calls.

### C09 — Environment-Dependent Safety Drift
Safety behavior changes based on debug/dev/staging environment variables.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BDB-Labs/aira-scanner](https://github.com/BDB-Labs/aira-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
