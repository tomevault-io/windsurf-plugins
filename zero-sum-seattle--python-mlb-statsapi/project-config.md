---
trigger: always_on
description: `python-mlb-statsapi` is an unofficial Python wrapper for the MLB Stats API.
---

# AGENTS.md

## Project overview

`python-mlb-statsapi` is an unofficial Python wrapper for the MLB Stats API.

The package provides synchronous access to MLB data such as:

* People and players
* Teams and rosters
* Schedules
* Games
* Box scores
* Play-by-play data
* Standings
* Venues
* Drafts
* Awards
* Player and team statistics

API responses are converted into Pydantic models with Python-style `snake_case` field names.

This package is not affiliated with Major League Baseball.

## Supported environment

The package supports:

* Python 3.10
* Python 3.11
* Python 3.12
* Poetry
* Pydantic v2
* Requests

Do not introduce a new runtime dependency unless the task clearly requires it and the benefit justifies increasing the package footprint.

## Repository structure

Important paths include:

```text
mlbstatsapi/
    mlb_api.py
    mlb_dataadapter.py
    mlb_module.py
    exceptions.py
    models/

tests/
    external_tests/
    fixtures/
    tools/
```

General responsibilities:

* `mlb_api.py` contains the public `Mlb` client and endpoint methods.
* `mlb_dataadapter.py` handles communication with the MLB Stats API.
* `exceptions.py` contains package exceptions.
* `models/` contains Pydantic models for MLB response data.
* `tests/external_tests/` contains tests that intentionally call the live MLB API.
* Offline tests must remain outside `tests/external_tests/`.

## Development setup

Install dependencies with:

```bash
poetry install
```

Run the complete test suite with:

```bash
poetry run pytest tests/
```

Run only offline tests with:

```bash
poetry run pytest tests/ --ignore=tests/external_tests
```

Run live MLB API tests with:

```bash
poetry run pytest tests/external_tests/
```

Build the package with:

```bash
poetry build
```

Run offline tests before submitting any change.

Run external tests when changing:

* MLB endpoint behavior
* Request parameters
* Response parsing
* Pydantic aliases
* Model field types
* Hydration behavior
* Schedule, game, roster, or statistics logic

## Testing rules

Do not rely exclusively on the live MLB API for test coverage.

Live API tests can fail because of:

* MLB API availability
* Data changing over time
* Seasonal data availability
* Undocumented response changes
* Historical games returning unusual payloads

Use deterministic mocked responses or recorded fixtures for behavior that does not require the live service.

Mocked HTTP tests must not be placed under `tests/external_tests/`.

Tests should verify actual values whenever possible.

Do not write tests that only use `hasattr()` on Pydantic models. A declared field can exist while silently remaining `None` because its alias does not match the MLB response key.

When fixing a reported game or player response, add a regression test that reproduces the original failure.

## Backward compatibility

Preserve the existing public API unless the task explicitly authorizes a breaking change.

Existing usage such as the following must continue to work:

```python
import mlbstatsapi

mlb = mlbstatsapi.Mlb()
player = mlb.get_person(664034)
```

Do not casually change:

* Public method names
* Public method arguments
* Return types
* Model attribute names
* Exported classes
* Exception inheritance
* Existing not-found behavior

The library currently maps many 404 responses to domain-level empty results such as:

* `None`
* `[]`
* `{}`

Do not change this behavior as part of unrelated work.

Any new package exception must inherit from:

```python
TheMlbStatsApiException
```

## MLB API model rules

The MLB Stats API is undocumented and inconsistent.

Do not assume every key follows ordinary camelCase rules.

Examples of unusual MLB capitalization may include:

```text
strikeOuts
groundOuts
calendarEventID
startTimeTBD
fullFMLName
aX
aY
aZ
```

Use explicit Pydantic aliases when the actual MLB response key cannot be derived safely.

When accepting multiple observed spellings, use Pydantic validation aliases such as `AliasChoices` rather than duplicating fields.

The base models intentionally ignore unknown fields to remain resilient to MLB API changes. Because of this, alias mistakes can silently discard data.

Tests for model changes must confirm that incoming values populate the intended model field.

Do not change a model field type based on one response without checking:

* Current data
* Historical data
* Missing values
* Null values
* Numeric strings
* Integer and floating-point variations

Prefer tolerant parsing when MLB returns inconsistent but logically equivalent values.

## HTTP adapter rules

All network calls must remain bounded.

Every HTTP request must use an explicit timeout.

Retries must be:

* Bounded
* Limited to safe HTTP methods
* Limited to transient failures
* Implemented with backoff
* Respectful of `Retry-After` when available

Do not retry ordinary client errors such as:

* 400
* 401
* 403
* 404

Do not retry Pydantic validation failures or JSON decoding failures.

Do not introduce default response caching without explicit approval. Live games, schedules, rosters, and historical data have different freshness requirements.

Do not introduce hardcoded global rate limiting without explicit approval.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zero-sum-seattle/python-mlb-statsapi](https://github.com/zero-sum-seattle/python-mlb-statsapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
