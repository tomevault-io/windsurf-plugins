---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of music management tools for handling live show collections and media synchronization. The codebase consists of:

1. **Phish Collection Tools** (`bin/`): Python scripts for comparing, renaming, and merging Phish live show collections
2. **Sync Scripts** (`sync/`): Bash scripts that use rsync to synchronize music collections between local storage, NAS backup, and Plex media server
3. **ZSH Functions** (`zsh/functions/`): Audio conversion and metadata utilities
4. **Borg Backup** (`bin/borg-backup`, `systemd/`): Bash script and systemd units for nightly incremental backup of the music library to a local NAS using Borg. Retention: 30 daily + 4 weekly + 12 monthly.

## Key Architecture Patterns

### Phish Collection Tools (`bin/`)

Three Python tools designed to work together as a pipeline when integrating a LivePhish torrent download with an existing collection:

- **`phish-rename`** — Queries livephish.com to rename show directories from any date-based format to `Phish-YYYY-MM-DD.Dot.Separated.Location.[LivePhishID]`. Requires `requests` and `beautifulsoup4`.
- **`phish-compare`** — Read-only diagnostic: shows matched/unmatched shows, studio album cross-references, and undated items in both the existing collection and torrent.
- **`phish-merge`** — Performs the actual merge in up to four phases: (1) rsync backup to NAS, (2) copy torrent-only shows, (3) replace matched shows with canonical torrent copies, (4) rename existing-only shows to torrent naming style. Supports `--dry-run` and `--phase N`.

All three tools share the same date-extraction logic and default paths:
- `DEFAULT_EXISTING = /data/media/Sorted/Unsorted/Music/Phish`
- `DEFAULT_TORRENT  = /data/torrents/Phish-Live.Phish.Project-2002-2026`

Python deps: `pip install -r requirements.txt` (`requests`, `beautifulsoup4`).

### Sync Script Pattern
All sync scripts follow a common pattern:
- Configuration via `sync/config/global.conf` (environment-specific paths) and per-artist `*.conf` files
- Studio album exclusion via `*-excludes.txt` files
- Interactive confirmation prompts before execution (skippable with `-y`)
- Support for dry-run mode via `-n` flag
- Common rsync options with progress reporting
- Two-phase sync: full backup to NAS, filtered sync to Plex

### Audio Processing Functions
- `flac2alac`: Converts FLAC files to ALAC format with tag correction
- `flacinfo`: Displays metadata for FLAC/ALAC files in tabular format

## Common Development Commands

### Phish Tools
```bash
# Rename new downloads to canonical format (dry run first)
bin/phish-rename /data/torrents/Phish-Live.Phish.Project-2002-2026
bin/phish-rename /data/torrents/Phish-Live.Phish.Project-2002-2026 --execute

# Compare existing collection vs torrent
bin/phish-compare

# Merge torrent into existing collection
bin/phish-merge --dry-run
bin/phish-merge --execute --nas /mnt/nas/Phish-backup

# Run only a specific phase
bin/phish-merge --phase 4 --dry-run
```

### Sync Scripts
```bash
# Unified sync command for all artists
./sync/music-sync phish              # Interactive sync with prompts
./sync/music-sync -n billy-strings   # Dry run preview
./sync/music-sync -y trey-anastasio  # Skip confirmation prompts
./sync/music-sync --help             # Show available artists and usage
```

### Borg Backup
```bash
# Run smoke test (uses temp dirs, safe anytime)
bin/borg-backup-test

# Check next scheduled run
systemctl list-timers borg-music-backup

# View last backup log
journalctl -u borg-music-backup.service -n 200

# List all archives
borg list /data/media-nas/Backups/borg

# One-time deployment (after cloning on a new machine)
# Edit ExecStart in systemd/borg-music-backup.service to match your checkout path
sudo cp systemd/borg-music-backup.{service,timer} /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now borg-music-backup.timer
```

### Audio Functions (from zsh/functions/)
```bash
flac2alac "Artist Name" "Album Name"   # Convert FLAC to ALAC with corrected tags
flacinfo *.flac                         # Display track information
```

## Testing

The repo has two test suites. All tests must pass before merging.

### Bats Tests (sync scripts — 28 tests)
```bash
cd tests
bats *.bats                    # Run all 28 bats tests
bats sync-lib.bats            # Unit tests (19 tests)
bats music-sync.bats          # Integration tests (9 tests)
bats --verbose-run *.bats     # Verbose output for debugging
```

### Pytest Tests (Python bin/ tools — 65+ tests)
```bash
pytest tests/                        # Run all Python tests
pytest tests/test_phish_rename.py    # Tests for phish-rename
pytest tests/test_phish_merge.py     # Tests for phish-merge
pytest -v tests/                     # Verbose output
```

### Running All Tests
```bash
cd tests && bats *.bats && cd .. && pytest tests/
```

### Test Architecture
- **Bats tests** — Each test runs in isolated temporary directories; rsync and ssh commands are mocked for safety; tests use actual config files with test data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ianchesal/music-management](https://github.com/ianchesal/music-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
