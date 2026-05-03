---
trigger: always_on
description: PCAP Hunter is an AI-enhanced threat hunting workbench for SOC analysts. It combines network analysis tools (Zeek, Tshark, PyShark) with LLMs and OSINT APIs to ingest, analyze, and extract actionable intelligence from PCAP files. Built with Streamlit for the web UI.
---

# CLAUDE.md — PCAP Hunter

## Project Overview

PCAP Hunter is an AI-enhanced threat hunting workbench for SOC analysts. It combines network analysis tools (Zeek, Tshark, PyShark) with LLMs and OSINT APIs to ingest, analyze, and extract actionable intelligence from PCAP files. Built with Streamlit for the web UI.

## Quick Reference

```bash
make install          # pip install -r requirements.txt
make run              # streamlit run app/main.py (port 8501)
make test             # PYTHONPATH=. pytest tests/ -v --cov=app
make lint             # ruff check .
make format           # ruff format .
make clean            # Remove caches
```

Always run tests with `PYTHONPATH=.` — this is required for absolute imports to resolve.

## Architecture

```
app/
├── main.py              # Streamlit entry point (session state, 8-tab UI)
├── config.py            # App defaults & constants (thresholds, paths)
├── analysis/            # Scoring, correlation, flow analysis, narration
├── database/            # SQLite case management (models.py, repository.py)
├── llm/                 # OpenAI-compatible API client (LM Studio supported)
├── pipeline/            # 10-stage analysis pipeline (pcap → report)
├── reports/             # PDF generation (WeasyPrint + Jinja2)
├── security/            # OPSEC hardening (sanitization, secure HTTP)
├── threat_intel/        # MITRE ATT&CK mapping engine
├── ui/                  # Streamlit components (layout, charts, config, cases)
└── utils/               # Shared utilities (export, crypto, network, YARA)
tests/                   # 22+ test modules, one per major component
docs/                    # EN + zh-TW user manuals, roadmap, test plan
data/                    # Runtime artifacts (carved/, zeek/, *.db) — gitignored
```

### 10-Stage Pipeline

1. `pcap_count.py` — Fast packet counting (tshark)
2. `pyshark_pass.py` — Deep packet parsing (up to 200K packets)
3. `zeek.py` — Automated Zeek execution and log parsing
4. `dns_analysis.py` — DGA detection, DNS tunneling, fast flux
5. `tls_certs.py` — TLS/SSL certificate chain validation
6. `beacon.py` — C2 beaconing detection (statistical analysis)
7. `carve.py` — HTTP payload extraction with SHA256 hashing
8. `yara_scan.py` — YARA rule-based file scanning
9. `osint.py` — Multi-provider OSINT enrichment (VT, AbuseIPDB, Shodan, etc.)
10. LLM synthesis — AI-powered threat report generation

## Tech Stack

- **Python 3.11+**, Streamlit 1.36+, Pandas 2.2+, NumPy 1.26+
- **Network**: Zeek, Tshark, PyShark 0.6+, Scapy 2.5+
- **LLM**: OpenAI SDK 1.30+ (LM Studio compatible)
- **OSINT**: VirusTotal, AbuseIPDB, GreyNoise, OTX, Shodan, MaxMind GeoIP
- **Security**: cryptography 42.0+ (PBKDF2 encrypted config)
- **Export**: WeasyPrint (PDF), STIX 2.0/2.1, ATT&CK Navigator, CSV/JSON
- **Dev**: Ruff (lint+format), Pytest 8.0+, pytest-cov, GitHub Actions CI

## Code Conventions

### Style

- **Formatter/Linter**: Ruff — line length 120, double quotes, 4-space indent
- **Lint rules**: E, F, I, W (see `pyproject.toml` for per-file ignores)
- **Type hints**: Used extensively; `from __future__ import annotations` for forward compat
- **Docstrings**: Google-style (Args/Returns sections) on public functions

### Naming

- Modules: `snake_case.py` (e.g., `ioc_scorer.py`, `dns_analysis.py`)
- Classes: `PascalCase` (e.g., `IOCScorer`, `ConfigManager`)
- Functions: `snake_case` (e.g., `rank_beaconing`, `validate_domain`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_DOMAIN_LENGTH`, `DATA_DIR`)
- Private: leading underscore (e.g., `_sanitize_for_llm`)

### Imports

- Always use **absolute imports**: `from app.pipeline.beacon import rank_beaconing`
- Standard library → third-party → local app modules (enforced by ruff `I` rule)
- Backward-compatible re-exports exist in `app/utils/common.py`

### Error Handling

- Custom exceptions inherit from `Exception` (e.g., `CarveError`)
- Use `logging` module, not print statements
- Streamlit phases use `phase.done()` for completion tracking

### Data Modeling

- Prefer `dataclass` for structured data
- Enums for fixed categories (e.g., `Severity`, `CaseStatus`)

## Testing

- **Framework**: Pytest with `--cov=app`
- **Location**: `tests/test_<module>.py` — one test file per major module
- **Run**: `make test` or `PYTHONPATH=. pytest tests/ -v --cov=app`
- **Pre-commit gate**: `make verify` — runs format check + lint + full test suite
- **PDF-focused**: `make test-pdf` — PDF generator, chart images, chart rendering, integration
- **Conventions**:
  - Test classes: `Test<Feature>` (e.g., `TestTechniqueMatch`)
  - Test functions: `test_<scenario>` (e.g., `test_periodicity_score_empty`)
  - Test both happy path and edge cases (empty, None, malformed)
  - Use dataclass instances for complex test objects
  - No shared conftest.py fixtures — tests are independent

### Testing discipline (non-negotiable)

**Before every commit, run `make verify`.** It must pass. CI runs the same
checks, so if `make verify` fails locally it will fail in CI.

**Use production-shape test data, not "looks reasonable" dicts.** If the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ninedter/pcap-hunter](https://github.com/ninedter/pcap-hunter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
