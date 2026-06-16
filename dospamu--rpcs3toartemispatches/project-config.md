---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo does

Converts RPCS3 emulator game patches (`patch.yml`) into Artemis PS3 cheat format (`.ncl` files) so they can be used on real PS3 hardware with Custom Firmware. All converted entries are marked with `(RPCS3)` in their cheat name and are **prepended** to each file so they appear first in Artemis.

## Running the converter

```bash
# Version-matched patches only → USERLIST/
node convert.js

# Also includes version-mismatched patches → USERLIST/ (labeled v01.XX)
node convert.js --risky
```

Both modes are idempotent: re-running skips entries already marked `(RPCS3)` in the target file. Results are logged to `conversion_report.json` (written next to `convert.js`, gitignored).

In risky mode, version-mismatched patches get the target version appended to their label: `Unlock FPS v01.04 (RPCS3)` so users know which game version the patch was written for. Both safe and risky output goes to `USERLIST/`.

## Tests

```bash
npm test                            # both suites
node scripts/convert.test.js        # converter only (nclVers, canonName, parseLine…)
node scripts/check_psxplace.test.js # monitor only (parsers, prependToNcl on temp files)
```

Plain `node:assert` scripts, no test framework. CI runs `npm test` before every scrape. When changing duplicate detection or version matching in `convert.js`, an end-to-end check is cheap and definitive: run `node convert.js` twice — the second run must report `Added 0 patch entries`.

## PSXPlace thread monitor (`scripts/check_psxplace.js`)

Daily automation (GitHub Actions, cron 06:00 UTC, `.github/workflows/check-psxplace.yml`) that scrapes PSXPlace thread #49905 via Camoufox (Cloudflare-resistant Firefox). It detects two kinds of activity:

- **New reply posts** → extracts Title IDs + NCL codes, prepends `Unlock FPS (PSXPlace)` entries to matching files in `PSXPlace Confirmed/` (>20 code lines in one post = quoted catalog, skipped).
- **First-post edits** (Joey85's catalog, detected via `first_post_hash`) → `parseFirstPost()` extracts structured entries (game name, TID, version) and can **create new files**.

Key behaviors:
- `prependToNcl()` returns `'added' | 'updated' | false`. Same cheat name with **different** codes replaces the old block in place (forum corrections); `[Tested]` blocks are **never** auto-replaced; identical entries are skipped.
- State (`known_posts.json`: `known_post_ids`, `first_post_hash`, `last_checked`) is saved **after** processing succeeds — a crash must not mark posts as known before their patches are extracted.
- Raw scraped content of new posts goes to `new_patches_raw/YYYY-MM-DD.txt`; `pr_body.txt` (gitignored) gates the workflow's commit step.
- The workflow commits results **directly to master** (`[Auto] PSXPlace new patches YYYY-MM-DD`).
- Empty `known_post_ids` = bootstrap mode: records all current posts without processing them.

## Patch label conventions

Label suffixes in cheat names indicate provenance and status:

| Label | Meaning | Author field |
|-------|---------|-------------|
| `(RPCS3)` | Official RPCS3 `patch.yml` | `RPCS3` or `FlexBy/RPCS3` etc. |
| `(PSXPlace)` | PSXPlace forum / PS3 Codes spreadsheet | Real person name (e.g. `FlexBy`, `vFxMz`, `illusion`) |
| `v01.XX (RPCS3)` | Version-mismatched RPCS3 patch | `RPCS3` |
| `[Tested]` suffix | Confirmed working on real PS3 hardware | unchanged |

Both `(RPCS3)` and `(PSXPlace)` types may coexist in the same `.ncl` file and may target different memory addresses. When a PSXPlace patch was also confirmed in RPCS3's database, prefer the `(RPCS3)` label to avoid duplicates.

## Folder structure

| Folder | Contents |
|--------|----------|
| `USERLIST/` | **2,542** Artemis `.ncl` files — full patch database (RPCS3 conversions + PSXPlace community patches) |
| `Working Artemis Patches/` | **39** personally-verified `.ncl` files — same games as `MAPI_PATCHES.md` |
| `PSXPlace Confirmed/` | **83** community-verified `.ncl` files from PSXPlace thread #49905 (Joey85) + Nascar1243 — grown automatically by the monitor |
| `beta_testing/` | Saved HTML exports of Discord beta-test channel (reference material for verifying patches) |

Patches confirmed on real PS3 hardware are marked `[Tested]` in the cheat name. Version-mismatched RPCS3 patches are labeled `v01.XX (RPCS3)` inline. `Working Artemis Patches/` and `PSXPlace Confirmed/` are curated subsets of `USERLIST/` shipped together in each release zip for users who only want known-good patches.

## File formats

### patch.yml (RPCS3 format)
Non-standard YAML — the file has **multiple `Anchors:` sections** scattered throughout, which breaks standard YAML parsers (`js-yaml` cannot load it). The custom line-by-line parser in `convert.js` handles this. `patch_new.yml` is a second copy at the same version — use `patch.yml` as the canonical source.

Structure:
```
Anchors:
  ANCHOR_NAME: &ANCHOR_NAME     # 2-space indent
    - [ type, 0xADDR, 0xVAL ]  # 4-space indent (anchors)

PPU-<hash>:                     # root level, keyed by PPU executable hash
  "Patch Name":                 # 2-space indent
    Games:
      "Game Title":

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DoSpamu/RPCS3toArtemisPatches](https://github.com/DoSpamu/RPCS3toArtemisPatches) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
