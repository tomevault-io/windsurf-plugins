---
trigger: always_on
description: CLI tool that fetches German legal data (laws and court decisions) from external
---

# oldp-ingestor

CLI tool that fetches German legal data (laws and court decisions) from external
sources and pushes them into an OLDP instance via its REST API.

## Quick reference

```bash
make install          # install into venv
make test             # run tests (100 tests, target coverage >= 90%)
make test-cov         # run with coverage report
make lint             # ruff check
make format           # ruff format
```

## Project layout

```
src/oldp_ingestor/
├── settings.py               # env config: OLDP_API_URL, OLDP_API_TOKEN, OLDP_API_HTTP_AUTH
├── client.py                 # OLDPClient — authenticated HTTP client for OLDP REST API
├── cli.py                    # argparse CLI: info, laws, cases, status subcommands
├── results.py                # result file writing and status dashboard
├── court_analysis.py         # court gap analysis from logs
├── providers/
│   ├── base.py               # Provider → LawProvider, CaseProvider (abstract bases)
│   ├── http_client.py        # HttpBaseClient — session, retry, pacing
│   ├── scraper_common.py     # ScraperBaseClient — HTML scraping helpers
│   ├── playwright_client.py  # PlaywrightBaseClient — browser automation
│   ├── de/                   # German providers
│   │   ├── ris_common.py     # RISBaseClient, extract_body, constants (shared HTTP)
│   │   ├── ris.py            # RISProvider(RISBaseClient, LawProvider) — legislation
│   │   ├── ris_cases.py      # RISCaseProvider(RISBaseClient, CaseProvider) — case law
│   │   ├── rii.py            # RiiCaseProvider — federal courts (rechtsprechung-im-internet.de)
│   │   ├── by.py             # ByCaseProvider — Bayern (gesetze-bayern.de)
│   │   ├── nrw.py            # NrwCaseProvider — NRW (NRWE database)
│   │   ├── ns.py             # NsCaseProvider — Niedersachsen (NI-VORIS)
│   │   ├── eu.py             # EuCaseProvider — EU courts (EUR-Lex)
│   │   ├── hb.py             # BremenCaseProvider — Bremen (5 SixCMS portals + PDF)
│   │   ├── sn_ovg.py         # SnOvgCaseProvider — Sachsen OVG Bautzen (PHP + PDF)
│   │   ├── sn.py             # SnCaseProvider — Sachsen ESAMOSplus (Playwright + PDF)
│   │   ├── sn_verfgh.py      # SnVerfghCaseProvider — Sachsen VerfGH (AJAX + PDF)
│   │   └── juris.py          # JurisCaseProvider + 10 state variants (juris-hosted portals)
│   └── dummy/                # Dummy providers (test/dev, not country-specific)
│       ├── dummy_laws.py     # DummyLawProvider (laws from Django fixture JSON)
│       └── dummy_cases.py    # DummyCaseProvider (cases from Django fixture JSON)
└── sinks/
    ├── base.py               # Sink ABC — write_law_book(), write_law(), write_case()
    ├── api.py                # ApiSink — wraps OLDPClient, delegates to .post()
    └── json_file.py          # JSONFileSink — writes JSON files to directory tree

tests/
├── test_providers.py          # provider unit tests (largest test file)
├── test_cli.py                # CLI integration tests
├── test_client.py             # OLDPClient tests
├── test_sinks.py              # sink unit tests

docs/
├── architecture.md            # class hierarchy, data flow, file layout
├── sinks.md                   # sink concept, CLI examples, custom sinks
├── politeness.md              # rate limiting, retry, User-Agent, cron
└── providers/
    ├── de/                    # German provider docs
    │   ├── ris.md             # RIS API endpoints, field mappings, request volumes
    │   ├── rii.md             # RII federal courts (rechtsprechung-im-internet.de)
    │   ├── by.md              # Bayern (gesetze-bayern.de)
    │   ├── nrw.md             # NRW (NRWE database)
    │   ├── ns.md              # Niedersachsen (NI-VORIS)
    │   ├── eu.md              # EUR-Lex (EU courts)
    │   ├── hb.md              # Bremen (5 SixCMS portals + PDF)
    │   ├── sn_ovg.md          # Sachsen OVG Bautzen (PHP + PDF)
    │   ├── sn.md              # Sachsen ESAMOSplus (Playwright + PDF)
    │   ├── sn_verfgh.md       # Sachsen VerfGH (AJAX + PDF)
    │   └── juris.md           # Juris state portals (10 variants, Playwright)
    └── dummy/
        └── dummy.md           # Dummy providers (test/dev fixture JSON)
```

## Provider class hierarchy

```
Provider                          (base.py — common root)
├── LawProvider                   get_law_books() -> list[dict], get_laws(code, date) -> list[dict]
│   ├── DummyLawProvider           loads from Django fixture JSON
│   └── RISProvider               fetches legislation from RIS API
└── CaseProvider                  get_cases() -> list[dict]
    ├── DummyCaseProvider          loads from Django fixture JSON
    ├── RISCaseProvider            fetches federal case law from RIS API
    ├── RiiCaseProvider            federal courts via rechtsprechung-im-internet.de
    ├── ByCaseProvider             Bavarian courts via gesetze-bayern.de
    ├── NrwCaseProvider            NRW courts via NRWE database
    ├── NsCaseProvider             Lower Saxony via NI-VORIS
    ├── EuCaseProvider             EU courts via EUR-Lex
    ├── BremenCaseProvider         Bremen (5 SixCMS court portals + PDF)
    ├── SnOvgCaseProvider          Sachsen OVG Bautzen (PHP site + PDF)
    ├── SnCaseProvider             Sachsen ESAMOSplus ordinary courts (Playwright + PDF)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openlegaldata/oldp-ingestor](https://github.com/openlegaldata/oldp-ingestor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
