---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## Scope and instruction priority

- These instructions apply to the entire repository.
- Follow direct user instructions first, then this file.
- Read `README.md` and the relevant code before changing behavior.
- Keep changes focused. Preserve unrelated work in the checkout.

## Project summary

LivePhish Downloader is a single-file Python command-line tool that uses a
visible Chrome session to download content available through a user's
authenticated LivePhish account. Selenium drives the site and reads Chrome
performance logs, `python-dotenv` loads optional local credentials, and
`requests` downloads recognized audio stream URLs. Selenium Manager resolves a
compatible ChromeDriver.

Repository: <https://github.com/jkowall/LivePhish-Downloader>

### Repository map

- `livephish_browser_downloader.py`: CLI, Selenium automation, stream URL
  capture, filename generation, checkpoints, and downloads
- `pyproject.toml`: canonical package, Python, dependency, and entry-point
  metadata
- `requirements.txt`: bounded runtime dependency compatibility manifest
- `.env.example`: safe template for optional automatic-login credentials
- `tests/`: standard-library unit tests for isolated helper behavior
- `README.md`: installation, usage, operating modes, and troubleshooting
- `CHANGELOG.md`: user-visible release history
- `CONTRIBUTING.md`: contributor setup and validation checklist
- `docs/MANUAL_SMOKE_TEST.md`: required live-browser validation checklist
- `.github/workflows/ci.yml`: cross-platform Python validation
- `LICENSE`: Apache License 2.0
- `.gitignore`: local environments, generated files, and downloads

`pyproject.toml` is the dependency source of truth. Keep the compatible bounds
in `requirements.txt` synchronized with it, and update installation or
troubleshooting documentation when dependencies change. The program must not
install packages or create an environment at runtime.

## Behavioral model

The CLI has three operating paths:

1. With no mode flag, the user logs in, opens one Stash item manually, and the
   script attempts to process every detected track. Login can use an ignored
   `.env` file or remain manual.
2. With `--all`, the script selects a Stash tab and processes every detected
   item. `--type` applies only to this path and defaults to `playlists`.
3. With `--interactive`, the user starts each track and confirms it in the
   terminal. Automatic single-item mode falls back to this workflow when it
   cannot detect tracks unless bounded automatic-selection flags are active.

Automated selection can be previewed with `--dry-run`, resumed from a 1-based
global position with `--start-at`, and capped with `--limit`. In `--all`, the
ordered track stream spans item boundaries. `--retry-failed` first selects
failed, expired, or unprocessed keys from
`<output>/.livephish-checkpoint.json`, then applies start and limit.

Important implementation constraints:

- Optional automatic login may read `LIVEPHISH_EMAIL` and
  `LIVEPHISH_PASSWORD` from the process environment or an ignored `.env` file.
- Enter credentials only on `https://id.livephish.com`. Never send them to a
  different host or over plain HTTP.
- Never print credentials, include them in exceptions, add real values to
  `.env.example`, or commit any `.env` variant other than `.env.example`.
- Preserve manual login as the fallback for missing credentials, MFA, rejected
  credentials, or identity-page changes.
- Quality behavior is explicit: `best` enables HiFi, prefers FLAC, and accepts a
  player fallback; `lossless` enables HiFi and fails or skips non-FLAC capture;
  `player` leaves the account setting unchanged and accepts the player stream.
  The default is `best`.
- Keep the first supported M4A or MP3 request as a fallback only when the quality
  policy permits it, and allow only a short grace period for FLAC so each track
  does not incur the full capture timeout.
- Never transcode a lossy source or describe it as lossless.
- Do not add authentication bypasses, DRM circumvention, or access to content
  outside the user's valid subscription.
- Stream capture must remain limited to validated HTTPS audio media hosts and
  paths. Preserve the case-insensitive exclusion for preview clips unless a
  verified behavior change requires otherwise.
- Never print or persist credentials, cookies, authorization headers, signed
  stream URLs, URL queries, or fragments. Checkpoints are URL-free.
- Existing destination files are skipped only after validating their audio
  signature. Invalid existing files must not be overwritten. Write new
  downloads to a same-directory `.part` path, remove failed or interrupted
  partials, and use an atomic replace only after completion.
- Dry runs must not start playback, download media, write checkpoints, or mark
  tracks successful.
- Keep checkpoints contained in the selected output directory and update them
  after each track result. `--retry-failed` requires an existing compatible
  checkpoint.
- LivePhish is a dynamic web application. Re-query DOM elements after
  navigation or playback changes instead of retaining stale Selenium elements.
- Selector changes should keep specific selectors ahead of broad fallbacks and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkowall/LivePhish-Downloader](https://github.com/jkowall/LivePhish-Downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
