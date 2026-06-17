---
trigger: always_on
description: Instructions for AI agents working on this codebase.
---

# CLAUDE.md

Instructions for AI agents working on this codebase.

## Project Overview

bharat-courts is an async Python SDK for accessing Indian court data. Two complementary backends + one federated facade:

1. **Live clients** (`hcservices`, `districtcourts`, `calcuttahc`, `judgments`, `sci`) — scrape the official eCourts portals. CAPTCHA-gated, rate-limited, can answer "current case status / cause list / orders in progress". This is the original SDK.
2. **Archive client** (`archive`, opt-in via `[archive]` extra) — DuckDB queries against the public AWS Open Data buckets (SCI 1950→present + 25 HCs, CC-BY-4.0). No CAPTCHA, no rate limits, but lags by 2–3 months. Used for historical research and bulk PDF retrieval.
3. **`Judgments` facade** (`facade.py`) — single entry point that owns both backends, picks the right one per query (CNR → archive; text → live; structured → archive; mixed → archive with title fallback), and returns a uniform `Judgment` list. This is the recommended default for "find a judgment matching X".

Coverage: 25+ High Courts, 700+ District Courts, the Supreme Court, plus the archive.

## Development Commands

```bash
# Install (requires Python 3.11+, use python3.12 if system python is older)
pip install -e ".[all]"

# Run unit tests (250 tests, no network needed)
pytest

# Run single test
pytest tests/test_hcservices_parser.py::test_parse_case_status_json

# Lint + format
ruff check .
ruff format --check .

# Auto-fix lint issues
ruff check --fix .
ruff format .

# Live integration tests — standalone scripts under tests/integration/.
# Not collected by pytest; invoke directly. See tests/integration/README.md.
python tests/integration/hcservices.py    # HC Services portal + CAPTCHA
python tests/integration/archive.py       # archive + facade vs real S3
python tests/integration/districtcourts.py
python tests/integration/calcuttahc_wpa_12886.py
```

## Architecture

### Source layout: `src/bharat_courts/`

- **`models.py`** — All data models. Dataclasses with `_Serializable` mixin providing `to_dict()` and `to_json()`. Key types: `Court`, `CaseInfo`, `CaseOrder`, `CauseListPDF`, `JudgmentResult`, `Judgment`, `SearchResult`. `Judgment` is the unified type used by the archive; `JudgmentResult` is the older type used by `JudgmentSearchClient`.
- **`courts.py`** — Static registry of all courts with eCourts state codes. `get_court("delhi")` returns a `Court` object. Also: `get_court_by_state_code("26")` (used by the archive to resolve HC partitions), `infer_court_from_cnr("DLHC...")` (4-letter prefix → Court, covers all 25 HCs + SCI). Codes verified against live portal.
- **`config.py`** — Pydantic Settings with `BHARAT_COURTS_` env prefix. Loaded once as module-level `config` singleton.
- **`http.py`** — `RateLimitedClient` wrapping httpx with retry, rate limiting, SSL bypass, and browser-like headers. Used by the live clients; the archive uses plain `httpx.AsyncClient` (no rate limiting needed for S3).
- **`captcha/`** — Pluggable CAPTCHA solving. `CaptchaSolver` ABC → `ManualCaptchaSolver` (stdin), `OCRCaptchaSolver` (ddddocr), `ONNXCaptchaSolver`. Only the live clients use these — the archive is CAPTCHA-free.
- **`hcservices/`** — Primary live client for hcservices.ecourts.gov.in (fully working).
- **`districtcourts/`** — Live client for services.ecourts.gov.in (700+ courts).
- **`calcuttahc/`** — Direct-website client for calcuttahighcourt.gov.in.
- **`judgments/`** — Live client for judgments.ecourts.gov.in.
- **`sci/`** — Live client for www.sci.gov.in (homepage feed only; case-no search not yet wired).
- **`archive/`** — Opt-in (`[archive]` extra). DuckDB over AWS Open Data parquet shards + per-tar / per-PDF caching. See "Archive module" below.
- **`facade.py`** — `Judgments` class, the federated find/fetch_pdf entry point. Owns lazy-initialised `ArchiveClient` + `JudgmentSearchClient`, routes by query shape. See "Federated facade" below.
- **`cli.py`** — Click CLI entry point. Command groups mirror SDK module names; the top-level `find` command is the CLI equivalent of `Judgments.find`.

### HC Services portal protocol

1. `GET main.php` → establishes `HCSERVICES_SESSID` cookie
2. `GET securimage/securimage_show.php` → CAPTCHA image (pinned to session)
3. `POST cases_qry/index_qry.php?action_code=showRecords` → case search (JSON response)
4. `POST cases_qry/index_qry.php` with `action_code=showCauseList` in body → cause list (HTML response)

Key details:
- `action_code` goes in **URL query string** for showRecords, but in **POST body** for showCauseList
- `rgyear` is **mandatory** for party name search — server returns `ERROR_VAL` if empty
- CAPTCHA is pinned to PHP session — must create a fresh session for each retry
- `fillCaseType` needs `court_code` (bench code), NOT `court_complex_code`
- Responses have BOM prefix (`\ufeff`) that must be stripped

### Response formats

- **showRecords** → JSON: `{"con":["[{...}]"], "totRecords":"N", "Error":""}`
  - `con[0]` is a JSON-encoded string of case records
  - Fields: `cino`, `case_no`, `case_no2`, `case_type`, `case_year`, `pet_name`, `res_name`
- **showCauseList** → HTML table with columns: Sr No | Bench | Cause List Type | View Causelist (PDF link)
- **fillHCBench / fillCaseType** → `code~name#` delimited text

### Error responses


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamshouvikmitra/bharat-courts](https://github.com/iamshouvikmitra/bharat-courts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
