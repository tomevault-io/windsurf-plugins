---
trigger: always_on
description: This file provides context for Claude Code when working on the olk project.
---

# CLAUDE.md

This file provides context for Claude Code when working on the olk project.

## What is this project?

`olk` is a CLI tool for Microsoft Outlook and OneDrive via the Microsoft Graph API. It provides terminal access to email, calendar, contacts, tasks, and OneDrive files for both personal Microsoft accounts and enterprise Azure AD/Entra ID accounts.

## Quick Reference

```bash
make build          # Build binary to ./bin/olk
make test           # Run tests
make lint           # Lint with golangci-lint
go mod tidy         # After changing dependencies
```

> **Validating on macOS:** running a freshly built `./bin/olk` against a real account triggers a macOS Keychain access prompt (each build is a new identity). A human must click **"Always Allow"** — you (an agent) can't dismiss the dialog, so don't treat a first-run hang as a bug; ask the user to approve it.

## Architecture

- **CLI framework**: `github.com/alecthomas/kong` — commands are Go structs with `Run(ctx *RunContext) error`
- **Auth**: Raw OAuth2 device code flow with PKCE (RFC 7636) against `login.microsoftonline.com` — no MSAL. Scopes defined in `internal/msauth/scopes.go`. Enterprise-only scopes (`MailboxSettings.ReadWrite`, `User.ReadBasic.All`) are only requested with `--enterprise` flag — personal accounts cannot consent to them. Token refresh is serialized per-email via `sync.Map` of mutexes to prevent race conditions
- **API**: Official `msgraph-sdk-go` wrapped in `internal/graphapi/` for ergonomic access
- **Secrets**: OS keyring via `github.com/99designs/keyring` (macOS Keychain, Linux Secret Service, Windows WinCred). File-backend password prompt writes to stderr (not stdout) to avoid corrupting piped output. Set `OLK_KEYRING_PASSWORD` for headless/non-interactive use
- **Output**: JSON envelope (`--json`), aligned table (default), TSV (`--plain`)
- **MCP server**: `olk mcp` (in `internal/cmd/mcp*.go`) runs a stdio Model Context Protocol server exposing a **curated** allowlist of read-first tools (`curatedTools` in `mcp_server.go`) — NOT the whole command tree. Tool calls reparse a rebuilt argv and run in-process with stdout captured under a mutex (`mcp_capture.go`). Read-only by default; `--allow-write <tool>` exposes a named curated safe-write tool (per-tool opt-in). No HTTP transport
- **Capability guards**: `--no-write`/`--no-send` are enforced once at the `graphapi.Client` layer (`ensureWritable`/`ensureMaySend`), so the guarantee holds across CLI, MCP, and scripts. `--enable-commands[-exact]`/`--disable-commands` gate dispatch via `commandAllowed()` (`commands.go`), checked in `Execute()` and reused to filter the MCP registry. `--wrap-untrusted` wraps `untrusted:"true"`-tagged struct fields in JSON/plain output (`internal/outfmt/untrusted.go`)
- **Timezone**: Display-layer conversion via `outfmt.ConvertTime()`. Resolved once per command via `RunContext.Timezone()` (flag > env > config > Local). JSON output emits UTC timestamps as RFC3339 with a `Z` suffix (normalized via `normalizeGraphUTC` — Graph's `DateTimeTimeZone.dateTime` strings lack a zone); envelope includes `timezone` field. IANA db embedded via `import _ "time/tzdata"`

## Key Patterns

- `RunContext` (in `internal/cmd/root.go`) lazily initializes the Graph client — auth commands skip it
- Graph SDK uses pointer types everywhere — always nil-check: `if x.GetFoo() != nil { *x.GetFoo() }`
- Each command is in its own file: `mail_list.go`, `mail_get.go`, etc.
- Desire paths in `desire_paths.go` delegate to real commands (e.g. `SendCmd` creates `MailSendCmd`)
- Config lives at `~/.config/olk/`, tokens in OS keyring keyed by `olk:token:<email>`

## Common Tasks

### Adding a new mail subcommand
1. Create `internal/cmd/mail_<name>.go` with the command struct and `Run` method
2. Add the struct to `MailCmd` in `internal/cmd/mail.go`
3. If needed, add the API method to `internal/graphapi/mail.go`

### Adding a new calendar subcommand
1. Create `internal/cmd/calendar_<name>.go` with the command struct and `Run` method
2. Add the struct to `CalendarCmd` in `internal/cmd/calendar.go`
3. If needed, add the API method to `internal/graphapi/calendar.go`

### Adding a new people subcommand
1. Create `internal/cmd/people_<name>.go` or add to `internal/cmd/people.go`
2. Add the struct to `PeopleCmd` in `internal/cmd/people.go`
3. If needed, add the API method to `internal/graphapi/people.go`

### Adding a new todo subcommand
1. Create `internal/cmd/todo_<name>.go` or add to `internal/cmd/todo.go`
2. Add the struct to `TodoCmd` in `internal/cmd/todo.go`
3. If needed, add the API method to `internal/graphapi/todo.go`

### Adding a new drive subcommand
1. Create `internal/cmd/drive_<name>.go` with the command struct and `Run` method
2. Add the struct to `DriveCmd` in `internal/cmd/drive.go`
3. If needed, add the API method to `internal/graphapi/drive.go`

### Adding a new flag to all commands
Add it to `RootFlags` in `internal/cmd/root.go` with `env:"OLK_*"` tag.

### Exposing a command as an MCP tool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rlrghb/olkcli](https://github.com/rlrghb/olkcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
