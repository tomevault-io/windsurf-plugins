---
trigger: always_on
description: This document provides strict rules and patterns for AI agents (Claude, Cursor, etc.) working on the Mock Jutsu ecosystem.
---

# Mock Jutsu — AI Development Guide

This document provides strict rules and patterns for AI agents (Claude, Cursor, etc.) working on the Mock Jutsu ecosystem.

**Developer:** Altan Sezer Ayan  
**GitHub:** https://github.com/altansayan  
**LinkedIn:** https://www.linkedin.com/in/altansezerayan/  
**Repository:** https://github.com/altansayan/mock-jutsu-api  
**License:** MIT — https://github.com/altansayan/mock-jutsu-api/blob/main/LICENSE

## 🛡️ STRICT DEVELOPMENT PROTOCOL (SOP)
The following 11-step lifecycle MUST be followed for every module. **No shortcuts allowed.**

1.  **Legal Check:** Ensure the data type is legal to mock. If it creates a real security vulnerability or is illegal, cancel the module immediately.
2.  **Unit Test (TDD) First:** Write unit tests in `tests/test_generators.py` based on real standards (ISO, IETF, etc.) BEFORE implementation.
3.  **Zero-Dependency Principle:** Use ONLY the Python Standard Library. Mathematical/cryptographic logic must be built from scratch in pure Python.
4.  **Code Development:** Implement the generator logic to pass the unit tests.
5.  **Integration (API) Tests:** Verify the data via the main API (`jutsu.generate()`) to ensure algorithmic compliance.
6.  **CLI & UI Tests:** Implement/test the CLI command and verify the new type appears in `mockjutsu list`.
7.  **Documentation Sync:** Run `generate_full_docs.py` to regenerate all 6 multilingual HTML files. Never edit HTML manually.
8.  **README.md Update:** Update the supported type counts, test counts, and UI badges in the main README.
9.  **Performance & Profiling:** Ensure latency is **< 1.5ms**. Test for CPU/RAM bottlenecks. Refactor if needed.
10. **Clean Code & DRY:** High readability, detailed docstrings, and modular architecture. Avoid spaghetti code.
11. **GitHub Push:** Only push to `main` after all 10 stages are successfully completed.

## 🌍 GLOBAL ECOSYSTEM STRATEGY (Fan-out)
Mock Jutsu aims to be the standard testing tool for all platforms:
- **PyPI (Python):** `pip install mockjutsu` (Active)
- **Homebrew (macOS/Linux):** `brew install mockjutsu`
- **NPM (JavaScript):** `npx mockjutsu` wrapper.
- **NuGet (.NET):** Standalone `.exe` via PyInstaller.
- **Maven (Java/Kotlin):** JNI/ProcessBuilder wrapper.
- **VS Code Marketplace:** Extension for direct IBAN/QR/UUID injection.

## ⚠️ MANDATORY RULES & GOTCHAS
- **NO UNAUTHORIZED CHANGES:** DO NOT change any file without asking the user first. Always ask for permission.
- **GITHUB MANDATE:** Every project produced by Altan Sezer Ayan MUST be uploaded to GitHub.
- **PYTHONPATH:** Always run tests with `$env:PYTHONPATH='src'` (PowerShell) or `export PYTHONPATH=src` (Bash).
- **Zero-Dependency:** Strict adherence. No external libraries.
- **Compliance:** Run `python scripts/audit_compliance.py` and `pytest --cov-fail-under=85` before every push.

---

## Project Overview

Mock Jutsu is a **zero-dependency, pure-Python** algorithmic mock data engine.
- **192 data types**, 6 locales (TR / US / UK / DE / FR / RU)
- All values are algorithmically generated (real checksums, ISO-compliant formats)
- Ships as a CLI tool (`mockjutsu`) and a Python API (`jutsu.generate()`)
- Python support: **3.10, 3.11, 3.12, 3.13** (3.9 is EOL — never add it back)
- Test coverage: **≥ 85%** enforced (currently ~97%)
- Performance: **< 1.5ms per call** (300ms for 200 iterations)

---

## Repository Layout

```
mock-jutsu-api/
├── src/mockjutsu/
│   ├── core.py                  # Master orchestrator — dispatches to generators
│   ├── cli.py                   # CLI commands + _REFERENCE table + _CAT_ORDER
│   ├── generators/
│   │   ├── identity.py          # TCKN, SSN, NIN, INN, SNILS...
│   │   ├── financial.py         # Cards (Luhn), IBAN, EMV QR, 3DS...
│   │   ├── banking.py           # SWIFT, SORT, routing, BIK...
│   │   ├── communication.py     # Phone, address, email, plate...
│   │   ├── meta.py              # UUID, JWT, IP, URL, API key, TOTP...
│   │   ├── corporate.py         # Company name, job title...
│   │   ├── health.py            # HL7, FHIR, DICOM, NHS, NPI, ICD-10...
│   │   ├── commerce.py          # VIN, vehicle, currency, invoice...
│   │   ├── iot.py               # RFID, NFC, NDEF, APDU, IR (NEC/RC5/Pronto)...
│   │   ├── barcode.py           # EAN-13, EAN-8, UPC-A, ISBN, GS1-128...
│   │   ├── telecom.py           # IMEI, ICCID, IMSI, MSISDN...
│   │   ├── financial_markets.py # ISIN, CUSIP, SEDOL, LEI, FIX, PSD2...
│   │   ├── crypto.py            # BTC/ETH addresses, tx hash, mnemonic...
│   │   ├── ecommerce.py         # SKU, order ID, tracking, DHL...
│   │   ├── location.py          # Lat/lon, timezone, country code...
│   │   ├── social.py            # Username, hashtag, bio, follower count...
│   │   ├── hardware.py          # Track2, EMV chip TLV, PIN block...
│   │   └── security.py          # CEF log, X.509 cert, pcap hex...
│   └── api/main.py              # FastAPI server (omitted from coverage)
│
├── tests/
│   ├── test_generators.py       # Main generator tests
│   ├── test_cli.py              # CLI command tests
│   ├── test_sync.py             # SYNC GUARD — enforces HTML/CLI/core consistency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altansayan/mock-jutsu-api](https://github.com/altansayan/mock-jutsu-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
