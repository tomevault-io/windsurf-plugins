---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

SpyEyes is a Python 3.10+ all-in-one OSINT CLI (`spyeyes` / `python -m spyeyes`).
10 subcommands: `ip` · `myip` · `phone` · `user` · `permute` · `whois` · `mx` · `email` ·
`subdomain` · `domain-emails` · `diff` · `history`. Bilingual (zh/en) — UI strings,
errors and **report contents** all switch with `--lang`. Current version: see
`__version__` in `spyeyes/__init__.py` (must stay in sync with `pyproject.toml`,
`docs/CHANGELOG.md` and the git tag — they are touched together in every release commit).

## Common commands

```bash
# Dev install (editable) + all dev tools
pip install -r requirements-dev.txt
pip install -e .            # registers `spyeyes` entry-point

# Run from source without install
python -m spyeyes <subcommand> ...

# Full lint suite (mirror of CI `lint` job)
ruff check .
mypy spyeyes tools/build_platforms.py --ignore-missing-imports
bandit -r spyeyes/ tools/ -ll      # -ll = MEDIUM+ severity only

# Tests (CI uses --timeout=15 --timeout-method=thread)
pytest tests/ -v
pytest tests/ --cov=spyeyes --cov-report=term-missing
pytest tests/test_spyeyes.py::TestTrackIp -v          # one class
pytest tests/test_spyeyes.py::TestTrackIp::test_ipv4_ok -v   # one test
pytest tests/ -k "subdomain"                          # name-substring filter

# Refresh bundled platforms.json from upstream (Maigret + Sherlock + WhatsMyName)
python tools/build_platforms.py
python tools/build_platforms.py --no-fetch --cache-dir .cache   # offline reuse
```

CI matrix lives in `.github/workflows/ci.yml`: `lint` (Python 3.14) gates `test`
across Linux × {3.10–3.14}, macOS × {3.10, 3.14}, Windows × {3.10, 3.14}.
Codecov uploads only from Ubuntu × 3.14.

## Architecture

### One-file core, intentionally
Almost everything lives in **`spyeyes/__init__.py`** (~7900 lines). It is organised
as labeled `# ===` sections in this order: CONFIG → I18N → Color → HTTP →
Country map → printing utils → IP / MyIP → Phone → Username (`Platform` NamedTuple
+ `PLATFORMS` curated list) → Permute → Recursive → WHOIS / MX / Email →
**Subdomain enum** → Diff → **Domain-email enum** → Output formatters → Report
generators (HTML/PDF/CSV/TXT/XMind/Graph) → CLI (`build_parser` / `run_cli` / `main`).

Don't split this file when adding features — the project deliberately keeps a
single import surface so users can `import spyeyes as sp` and get every symbol.
`spyeyes/__main__.py` is a 2-line re-export so `python -m spyeyes` works.

### Entry points
- `main()` (line ~7866) — forces stdout/stderr to UTF-8 (Windows cp936 would crash
  on emoji), parses args, picks language, then dispatches to:
- `run_cli(args)` (line ~7516) — every subcommand (CLI mode)
- `menu_loop()` (interactive menu) — when no subcommand given
- `build_parser()` — single source of truth for CLI flags + bilingual `epilog`

### Plugin-style source registries
Two dicts make new OSINT data sources a one-line addition:
- `SUBDOMAIN_SOURCES` (~line 2589) → 6 passive sources (`crt.sh`, CertSpotter,
  HackerTarget, OTX, Wayback, optional subfinder). Each `_src_*` returns a `set[str]`.
- `DOMAIN_EMAIL_SOURCES` (~line 3587) → 6 passive sources (`crt.sh`, WHOIS, Bing,
  DDG, Wayback, GitHub). Each `_emails_from_*` returns a `set[str]`.

Both are fanned out via `ThreadPoolExecutor(max_workers=len(...))` and **silently
swallow per-source failures** (returns empty set + records error name in `_stats`).
Don't add cross-source coupling — the design contract is "any one source down,
the rest still work". `bandit.skips=["B110","B112"]` in `pyproject.toml` exists
for this reason; do **not** add new bare `except: pass` outside this OSINT-source pattern.

### Report generation (`--save`)
`_maybe_save(target, prefix, data)` (line ~4926) dispatches by file suffix:
- `.json` (default) / `.md` / `.html` / `.pdf` (needs `spyeyes[pdf]`) / `.txt` /
  `.csv` / `.xmind` / `.graph.html`
- `.graph.html` must be checked **before** `.html` (else the generic `.html`
  branch wins). See the existing dispatch — preserve order if you add formats.
- Directory target (ends with `/` or existing dir) always writes JSON with
  `<prefix>_<timestamp>.json`.
- All HTML/Graph/XMind paths go through `_html_escape`; CSV cells go through
  `_csv_safe` (prefixes `'` to defang `= + - @ \t \r` formula injection).
- Every report function honours `get_lang()` so `--lang en` and `--lang zh`
  produce different headings/labels (including CSV headers).

### i18n
`TRANSLATIONS` dict at top (line ~232) is `{lang: {key: template}}`. Use
`t('some.key', name=value)` everywhere user-facing — never hard-code Chinese or
English. New strings need both `'en'` and `'zh'` entries.

### State & config
- `~/.spyeyes/config.json` — persisted UI language
- `~/.spyeyes/history.jsonl` — per-query metadata only (no full results); disable
  with env `SPYEYES_NO_HISTORY=1`
- `~/.spyeyes/env` — KEY=VALUE API keys, auto-loaded by `_load_env_file()` at
  import time. Shell-exported vars take precedence (the file only fills gaps).
  Recognised keys: `SPYEYES_OTX_API_KEY`, `SPYEYES_CERTSPOTTER_API_KEY`,
  `PDCP_API_KEY`, `SPYEYES_GITHUB_TOKEN`, `SPYEYES_REPORTS_DIR`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Akxan/SpyEyes](https://github.com/Akxan/SpyEyes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
