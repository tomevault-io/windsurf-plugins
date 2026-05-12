---
trigger: always_on
description: This file provides guidance to AI coding agents working with this repository.
---

# hey-cli

This file provides guidance to AI coding agents working with this repository.

## What is hey-cli?

hey-cli is a CLI and TUI interface for [HEY](https://hey.com). 
It allows users to read and send emails, manage their boxes, manage their calendars and journal entries.
The TUI is primarily intended for human use, while the CLI is primarily intended for use by AI agents and for scripting.

## Development commands

This project uses make.

```bash
make build   # Builds the project into a binary located at ./bin/hey
make test    # Runs the tests
make lint    # Lints the code
make clean   # Cleans the build artifacts
make install # Installs the binary to /usr/local/bin/hey
```

## Architecture Overview

This is a Go project that uses:
- [spf13/cobra](github.com/spf13/cobra) for the CLI interface
- [charm.land/bubbletea/v2] for the TUI interface along with bubbles/v2 and lipgloss/v2 (these are new versions that recently came out and differ from the v1 versions!)

All API interactions go through the HEY SDK (`hey-sdk/go`), with typed service methods accessed via `internal/cmd/sdk.go` (e.g., `sdk.Boxes().List`, `sdk.Messages().Create`, `sdk.Calendars().GetRecordings`). Authentication and token refresh are handled via `internal/auth/`.

### Authentication

Authentication supports four methods, all managed through `internal/auth/`:

1. **Browser-based OAuth with PKCE** (primary) — `hey auth login` opens a browser for OAuth authentication against HEY's own OAuth server (`/oauth/authorizations/new`), using PKCE (S256) for security. A local callback server on `127.0.0.1:8976` receives the authorization code, which is exchanged for access and refresh tokens at `/oauth/tokens`.
2. **Pre-generated bearer token** — `hey auth login --token TOKEN` stores a token directly.
3. **Browser session cookie** — `hey auth login --cookie COOKIE` uses an existing HEY.com session.
4. **Environment variable** — Set `HEY_TOKEN` to use a token without storing it.

The auth Manager (`internal/auth/auth.go`) proactively refreshes tokens with a 5-minute expiry buffer. The SDK uses the Manager to authenticate requests via a bridge in `internal/cmd/sdk.go`.

All data-access commands call `requireAuth()` before making API calls. Auth subcommands (`hey auth login`, `hey auth logout`, `hey auth status`) work without authentication.

### State storage

Configuration (base URL only) is stored in `~/.config/hey-cli/config.json`. Credentials are stored in the system keyring (service name: `hey`) with automatic fallback to `~/.config/hey-cli/credentials.json` when the keyring is unavailable. Set `HEY_NO_KEYRING=1` to force file storage.

### CLI

Remember to update the examples in the README when you change, add or remove CLI commands.

### HTML content

Some HEY API endpoints return 204 or incomplete data via JSON, but the full HTML content is available by scraping the edit page (e.g., `/calendar/days/{date}/journal_entry/edit` contains the Trix editor hidden input with full HTML). When an API endpoint returns incomplete data, check the corresponding web page for the full content. The `internal/htmlutil` package provides `ToText` (HTML→plain text), `ExtractImageURLs`, and `ParseTopicEntriesHTML` shared by both CLI and TUI. HEY uses Trix editor with `<figure data-trix-attachment="{...}">` for attachments — image URLs in those attributes are relative paths requiring authentication via `sdk.Get`.

### TUI structure

The TUI uses the `sectionView` interface pattern. Each top-level section (Mail, Calendar, Journal) implements `sectionView` and owns its data, fetch commands, key handling, rendering, and help bindings. The main model delegates to the active view.

| File | What it contains |
|------|-----------------|
| `section_view.go` | `sectionView` interface + `viewContext` shared dependencies |
| `tui.go` | Core model, `Update` router, `View`, key routing, shared utilities |
| `mail.go` | `mailView` — boxes, postings, topic threads, posting actions, entry rendering |
| `calendar.go` | `calendarView` — calendars, recordings, recording detail |
| `journal.go` | `journalView` — date navigation, journal entries |
| `nav.go` | Header rendering, section/box/calendar/journal nav items, shortcuts |
| `content.go` | `contentList` (posting list) and `recordingList` (recording list) |
| `loading.go` | Hourglass loading animation |
| `styles.go` | Colors, lipgloss styles, error display |
| `help.go` | Help bar at screen bottom |
| `kitty.go` | Kitty graphics protocol for inline images |
| `html.go` | Thin wrappers around `htmlutil` |

To add a new section: implement the `sectionView` interface in a new file, add a field and constructor call in `newModel`, and add a case in `switchSection`.

### Inline images in the TUI

The TUI renders inline images using the Kitty graphics protocol's Unicode Placeholder extension (`internal/tui/kitty.go`). This works because Bubble Tea's cell-based renderer corrupts raw APC escape sequences, but Unicode placeholders are regular text that survives rendering. The approach has three steps:

1. **Upload** — image data is sent to the terminal via `tea.Raw()` with `a=t` (transmit only) and `q=2` (suppress response), then a virtual placement is created with `U=1`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basecamp/hey-cli](https://github.com/basecamp/hey-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
