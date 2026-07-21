---
trigger: always_on
description: This file provides context for LLM assistants (Claude Code and similar tools) working in this repository.
---

# LLM Assistant Guide for `market-prices` package
This file provides context for LLM assistants (Claude Code and similar tools) working in this repository.

In all context files, a '@' prefixing a path indicates that the path is defined relative to the project root in which this `AGENTS.md` file is located.

## Skills

Identify all available skills in the @.agents\skills directory

## LLM context

Add the 'agents' label to any PR that amends:
- this @AGENT.md
- any SKILL.md file

## Project Overview

**market-prices** is a Python package to create meaningful OHLCV datasets for financial instruments. It provides for enchanced querying and post-processing of financial price data.

See @pyproject.toml for project metadata and dependencies.

### Repository Layout

```
market_prices/
├── .agents/                            # instructions for LLM coding agents
│   └── skills/                         # skills for LLM coding agents
│       ├── dependencies-management/
│       │   └── SKILL.md
│       └── update-agents-md/
│           └── SKILL.md
├── .github/
│   ├── workflows/
│   │   ├── build-test.yml
│   │   ├── draft-release-notes.yml
│   │   └── release.yml
│   ├── dependabot.yml
│   └── release-drafter.yml
├── docs/
│   ├── developers/
│   │   ├── other_internals.md          # notes on non-obvious internal design decisions
│   │   ├── releases.md
│   │   ├── serving_data.md
│   │   ├── testing.md
│   │   └── typing_doc.md               # typing conventions and documentation style
│   ├── media/
│   │   ├── readme_pt.png               # screenshot of .pt accessor used in README
│   │   └── splash.png
│   ├── public/
│   │   ├── method_doc.md
│   │   ├── parsing.md
│   │   └── typing.md
│   ├── tutorials/
│   │   ├── resources/                  # CSV data files for tutorial notebooks
│   │   ├── anchor.ipynb
│   │   ├── data_availability.ipynb
│   │   ├── intervals.ipynb
│   │   ├── other_get_options.ipynb
│   │   ├── other_prices_methods.ipynb
│   │   ├── periods.ipynb
│   │   ├── prices.ipynb
│   │   ├── pt_accessor.ipynb
│   │   ├── quickstart.ipynb
│   │   └── specific_query_methods.ipynb
│   └── tutorials_docs.md
├── src/
│   └── market_prices/                  # `PricesYahoo`, `PricesCsv`
│       ├── prices/                     # price-serving classes
│       │   ├── config/                 # configuration data for price sources
│       │   │   └── config_yahoo.py     # includes mappings for Yahoo data
│       │   ├── base.py                 # Core logic in `PricesBase`
│       │   ├── csv.py                  # `PricesCsv`: prices from local CSV files
│       │   └── yahoo.py                # `PricesYahoo`: prices via yahooquery
│       ├── support/                    # support for tutorials and tests
│       │   └── tutorial_helpers.py     # identify data to use in tutorials and tests
│       ├── utils/                      # utility modules
│       │   ├── calendar_utils.py       # includes `CompositeCalendar`
│       │   ├── general_utils.py        # general utils
│       │   └── pandas_utils.py         # pandas-specific utilities and context managers
│       ├── data.py                     # data administrator (by base interval)
│       ├── daterange.py                # derive date ranges for price requests
│       ├── errors.py                   # custom exception classes
│       ├── helpers.py                  # helpers (project-specific)
│       ├── intervals.py                # `TDInterval`, `DOInterval`, `BI` and helpers
│       ├── mptypes.py                  # custom types and aliases
│       ├── parsing.py                  # validates and coerces public input parameters
│       └── pt.py                       # .pt pandas accessor for custom DataFrame ops
├── tests/
│   ├── resources/                      # HDF5 stores and CSV fixtures
│   ├── conftest.py
│   ├── hypstrtgy.py                    # Hypothesis strategies
│   ├── test_base.py
│   ├── test_base_prices.py
│   ├── test_calendar_utils.py
│   ├── test_csv.py
│   ├── test_data.py
│   ├── test_daterange.py
│   ├── test_helpers.py
│   ├── test_intervals.py
│   ├── test_limits.py                  # tests for behaviour at data availability limits
│   ├── test_mpst.py                    # tests for complex Hypothesis strategies
│   ├── test_mptypes.py
│   ├── test_pandas_utils.py
│   ├── test_parsing.py
│   ├── test_pt.py
│   ├── test_tutorial_helpers.py
│   ├── test_yahoo.py
│   └── utils.py
├── .pre-commit-config.yaml
├── .python-version
├── AGENTS.md
├── CLAUDE.md
├── LICENSE.txt
├── MANIFEST.in
├── README.md
├── mypy.ini
├── pyproject.toml
├── pytest.ini
├── requirements.txt
├── ruff.toml
└── uv.lock
```

## Technology Stack

| Category | Tools |
|---|---|
| Python | 3.10–3.14 (`.python-version` pins 3.14) |
| Package manager | `uv` |
| Build backend | `setuptools` + `setuptools_scm` |
| Testing | `pytest` |
| Linting/formatting | `ruff` |
| Type checking | `mypy` |
| Git hooks | `pre-commit` |
| Data Manipulation | `pandas`, `numpy` |
| Calendars of Market Hours | `exchange-calendars` |

The current project version is managed by `setuptools_scm` and written to `src/market_prrices/_version.py`.
IMPORTANT: `src/market_prices/_version.py` is auto-generated and you should not edit it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maread99/market_prices](https://github.com/maread99/market_prices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
