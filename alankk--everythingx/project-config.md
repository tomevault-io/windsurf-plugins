---
trigger: always_on
description: EverythingX is a fast file-name search tool for macOS and Linux, inspired by [Everything by Voidtools](https://www.voidtools.com/support/everything/). It consists of three binaries:
---

# EverythingX — Agent Instructions

## Project Overview

EverythingX is a fast file-name search tool for macOS and Linux, inspired by [Everything by Voidtools](https://www.voidtools.com/support/everything/). It consists of three binaries:

| Binary | Source | Purpose |
|---|---|---|
| `everythingxd` | `cmd/service/` | Background daemon — indexes the filesystem into SQLite via FSEvents (macOS) or fanotify (Linux) |
| `everythingx` | `cmd/everythingx/` | GUI app — Fyne-based search interface |
| `ev` | `cmd/cli/` | CLI tool — fast command-line search |

## Tech Stack

- **Language**: Go 1.23+
- **GUI**: [Fyne v2](https://fyne.io/) (`fyne.io/fyne/v2`) with `github.com/dweymouth/fyne-tooltip`
- **Database**: SQLite3 via `github.com/mattn/go-sqlite3` (CGO required)
- **FS Events (macOS)**: `github.com/fsnotify/fsevents` (FSEvents API, `//go:build darwin`)
- **FS Events (Linux)**: `golang.org/x/sys/unix` fanotify with `FAN_MARK_FILESYSTEM` (`//go:build linux`, requires root + kernel 5.9+)
- **CLI flags**: `github.com/jessevdk/go-flags`

## Repository Layout

```
cmd/
  service/        # everythingxd daemon
    common.go           # shared daemon code (no build tag)
    main_darwin.go      # macOS FSEvents monitoring (//go:build darwin)
    main_linux.go       # Linux fanotify monitoring (//go:build linux)
    common_test.go      # platform-agnostic tests
    main_darwin_test.go # macOS-only tests
    main_linux_test.go  # Linux-only tests
    everythingxd.service # systemd unit file
    com.github.alankk.everythingxd.plist # launchd plist (macOS)
  everythingx/    # GUI app (main.go, ui.go, theme.go, assets.go, open_darwin.go, open_linux.go)
  cli/            # ev CLI tool
internal/
  ffdb/           # SQLite database package (all DB operations)
  shared/         # Models (SearchResult, EventRecord) and utilities
tools/            # Developer utilities (benchmarks, disk scan, etc.)
e2eTest/          # End-to-end tests
assets/           # App icons, screenshots, everythingx.desktop (Linux)
package/          # macOS .pkg installer assets + Linux nfpm scripts
bin/              # Compiled binaries (git-ignored)
nfpm.yaml         # .deb/.rpm packaging config (Linux)
install.sh        # Universal curl|sh installer (downloads latest release)
install-local-macos.sh # macOS install from unpacked files (launchd)
install-local-linux.sh # Linux install from unpacked files (systemd)
uninstall.sh      # macOS uninstall script
uninstall-linux.sh # Linux uninstall script
```

## Architecture

### Data Flow

1. **`everythingxd`** (service):
   - On startup, performs an initial full-disk scan and populates the SQLite DB.
   - **macOS**: subscribes to FSEvents for real-time create/delete notifications.
   - **Linux**: opens a fanotify fd with `FAN_MARK_FILESYSTEM` for mount-level monitoring.
   - Writes to the DB, which is opened in WAL mode to allow concurrent readers.
   - `shouldIgnorePath()` is defined per-platform: macOS skips `/System/Volumes/Data`; Linux skips `/proc`, `/sys`, `/run`, `/dev`, `/snap`.

2. **`everythingx` / `ev`** (consumers):
   - Open the DB **read-only** (`file:path?mode=ro`).
   - Execute prefix/substring searches via `ffdb.PrefixSearch`.

### Database Schema

```sql
CREATE TABLE files (
    filename    TEXT NOT NULL,
    fullpath    TEXT NOT NULL UNIQUE,
    event_id    INTEGER,
    object_type INTEGER
);
CREATE INDEX idx_filename ON files(filename COLLATE BINARY);
```

Default DB path: `/var/lib/everythingx/files.db`

### Search Query

```sql
SELECT fullpath, object_type
FROM files
WHERE filename LIKE ? COLLATE BINARY
ORDER BY filename ASC
LIMIT ?
```

The `%term%` wildcard is prepended by `PrefixSearch` in `internal/ffdb/ffdb.go`.

## Key Packages

### `internal/ffdb`

All database logic lives here:

- `CreateDB(pathname)` — creates a new DB with schema + indexes
- `OpenDB(pathname)` — opens for read/write (service)
- `OpenDBReadOnly(pathname)` — opens read-only (GUI/CLI)
- `PrefixSearch(prefix, limit)` — returns `[]*shared.SearchResult`
- `InsertRecord(record)` / `DeleteRecord(fullpath)` — called by the service

Prepared statements (`prefixSearchStmt`, `insertStmt`, `deleteStmt`) are package-level globals initialized on open.

### `internal/shared`

- **`SearchResult`** — `{Fullpath string, ObjectType ObjectType}`
- **`EventRecord`** — `{Filename, Path, ObjectType, EventID, EventTime, FoundOnScan}`
- **`ObjectType`** — `ItemIsFile`, `ItemIsDir`, `ItemIsSymlink`, etc.
- `FileExists(path)` — stat-based existence check
- `GetFileSizeMod(path)` — returns human-readable size + mod time
- `SplitFileName(filename, term)` — splits for bold-highlighting

### `cmd/everythingx` (GUI)

- Built with Fyne; uses `fyne.io/fyne/v2/widget.Table` for results.
- `handleAutoCompleteEntryChanged` debounces and triggers search on every keystroke.
- Double-click or Enter on a result opens the file's containing directory: `open -R` on macOS (`open_darwin.go`), `xdg-open` on Linux (`open_linux.go`).
- Theme switching supported via `theme.go`.
- Max results capped at `maxSearchResults = 1000`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlanKK/everythingx](https://github.com/AlanKK/everythingx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
