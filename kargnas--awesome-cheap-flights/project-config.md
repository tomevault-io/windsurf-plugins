---
trigger: always_on
description: - Tone: Concise English for UI.
---

# Repository Guidelines

<writing_style>

- Tone: Concise English for UI.
- Use 4-5 examples.
- Rule: Max 12 words per sentence.
- Rule: Time info in minutes.
- Rule: End each sentence with a period.
- Example: Summary delayed. Retry in about 5 min.
  </writing_style>

<uxui_writing_style>
- Audience: End-user product surfaces only, never Codex responses.
- Tone: Natural UI concise English (Option 4 selection).
- Structure: Short clauses, ≤12 words, period at end.
- Time references: Express in minutes.
- Example 1: Summary pending. Retry in 5 min.
- Example 2: Seating check running. Wait 3 min.
- Example 3: Refund in progress. Up to 10 min.
- Example 4: Fare refresh active. Needs 4 min.
- Example 5: Boarding pass queue. Notice in 2 min.
</uxui_writing_style>

## Project Structure & Module Organization

- `awesome_cheap_flights/cli.py`: CLI arguments + config loader for console script and uvx runs; direct module execution now exits via `SystemExit(main())`.
- `awesome_cheap_flights/__main__.py`: Enables `python -m awesome_cheap_flights` compatibility.
- `awesome_cheap_flights/pipeline.py`: Core scraping/search pipeline including data structures, HTML parsing, and CSV export utilities; plan legs are ordered by `departures` keys and hidden legs only bridge connected adjacent legs.
- `sample.config.yaml`: Minimal example config used for smoke tests; copy to `config.yaml` for local overrides.
- `output/`: Git-ignored directory for generated CSV files (only tracked when explicitly whitelisted).

## Logging & Observability

- Pipeline logs now flow through a shared Rich console with color-coded warnings and errors.
- `run_search` drives a Rich progress bar (TTY only) that advances per itinerary pair and logs row deltas.
- Non-interactive sessions fall back to concise text logs on stderr while preserving the same counts.
- A summary table prints after each run with processed itineraries, skips, rows collected, and elapsed minutes.
- A search overview table is emitted upfront listing origin codes, destination codes, pax, currency, and other knobs; each iteration label repeats route, pax, and currency for quick scanning.
- Progress labels log a concise flight summary (best fare, airline, leg).
- Rows deduplicate identical flights per journey leg to avoid repeated segments.
- Each plan saves an `<csv_stem>_itineraries.xlsx` workbook (default unlimited per leg; clamp via `leg_limit`).
- Workbook leg exports now include `origin_code` and `destination_code` columns.
- `stop_notes` now include arrival code plus layover code/city/duration when available.
- Missing prices render as `N/A` in CSV and workbook exports.
- CLI surfaces itinerary limits and recommended values when exporting workbooks.
- The progress bar announces `Ctrl+C`; interrupting saves `draft-<original>.csv` and reports remaining itineraries.
- Legs spanning more than seven departure days emit an extreme warning because those runs can last hours and invite Google anti-abuse throttling.

## Build, Test, and Development Commands

- `uvx awesome-cheap-flights@latest --help`: Smoke-check the published CLI.
- `uv run python -m awesome_cheap_flights.cli --config sample.config.yaml --output output/demo.csv`: Executes a lightweight search using the sample config.
- `uv run python -m awesome_cheap_flights.cli --output output/full.csv --departure ICN --destination FUK --outbound 2026-01-01 --inbound 2026-01-04`: Direct CLI run overriding config-specified values.
- `uv lock`: Refreshes dependency lockfile after editing `pyproject.toml`.
- Run uv lock after dependency bumps to refresh typing_extensions pin.
- Add `--debug` to CLI runs when you need the full provider error payload.

## Coding Style & Naming Conventions

- Python 3.10+, PEP 8 defaults; prefer 4-space indentation and descriptive snake_case identifiers.
- Keep CLI/YAML keys in lowercase plural form (`departures`, `destinations`, `itineraries`).
- YAML comment style must use `#`; inline annotations go after the IATA code (e.g., `ICN # Seoul Incheon`).
- Rich-powered logging is part of the packaging footprint; keep `rich>=13.7` declared in `pyproject.toml` when adjusting deps.
- CLI accepts comma-separated inputs (e.g., `--departure ICN,GMP`) and the legacy `Name:CODE` syntax has been removed.
- Use `--outbound`/`--inbound` to override itinerary dates from the CLI (single dates or `start:end` ranges expand all combinations).

## Testing Guidelines

- No automated tests yet; validate changes by running the sample config and inspecting generated CSV size and headers.
- When adding parsing logic, craft targeted configs in `sample.config.yaml` to reproduce new branches before committing.

## Commit & Pull Request Guidelines

- Follow existing concise, imperative commit style (e.g., “Switch to departures/arrivals config with pure IATA codes”).
- Each PR should include: summary of changes, CLI command used for verification, any config updates, and links to related issues.
- Keep CSV artifacts out of version control unless required for review; share sample outputs via temporary paths or gists.
- ALWAYS update README.md and AGENTS.md whenever you touch project logic. After your edits run `git rev-parse HEAD` and replace the final `Last commit id:` line in both files with that hash (exactly once per file).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kargnas/awesome-cheap-flights](https://github.com/kargnas/awesome-cheap-flights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
