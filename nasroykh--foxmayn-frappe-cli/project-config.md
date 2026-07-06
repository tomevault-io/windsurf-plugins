---
trigger: always_on
description: **ffc** (Foxmayn Frappe CLI) — A Go CLI for interacting with Frappe ERP sites via the REST API.
---

# CLAUDE.md — Project Context for AI Agents

## Project

**ffc** (Foxmayn Frappe CLI) — A Go CLI for interacting with Frappe ERP sites via the REST API.

## Tech Stack

- **Language:** Go 1.25
- **CLI framework:** [cobra](https://github.com/spf13/cobra)
- **Config:** [viper](https://github.com/spf13/viper) (YAML + env vars)
- **HTTP client:** [resty](https://github.com/go-resty/resty)
- **Table & styling:** [lipgloss v2](https://charm.land/lipgloss/v2) + built-in `table` sub-package
- **Forms & prompts:** [huh](https://github.com/charmbracelet/huh)
- **Spinner:** [huh/spinner](https://github.com/charmbracelet/huh) (standalone, no bubbletea loop needed)
- **MCP server:** [mark3labs/mcp-go](https://github.com/mark3labs/mcp-go) v0.46.0 (stdio + StreamableHTTP transports)

## Build & Run

```bash
make build          # Compile binary to ./bin/ffc
make install        # Install to $GOPATH/bin + set up config
make tidy           # go mod tidy
make vet            # go vet ./...
make fmt            # gofmt -w .
make clean          # Remove binary
```

Version info is injected at build time via ldflags (see Makefile).

## Release

Releases are automated via GoReleaser + GitHub Actions (`.github/workflows/release.yml`).

To cut a release:

```bash
git tag v0.1.0
git push origin v0.1.0
```

GitHub Actions will cross-compile for linux/darwin/windows × amd64/arm64, create a GitHub Release, upload the tarballs, and generate `checksums.txt`.

End users install with:

```bash
# Linux / macOS
curl -fsSL https://raw.githubusercontent.com/nasroykh/foxmayn_frappe_cli/main/install.sh | sh

# Windows (PowerShell or cmd.exe)
powershell -ExecutionPolicy Bypass -Command "irm https://raw.githubusercontent.com/nasroykh/foxmayn_frappe_cli/main/install.ps1 | iex"
```

**Key files:**
- `.goreleaser.yaml` — build matrix, archive naming, checksum config
- `.github/workflows/release.yml` — triggers on `v*` tags, runs GoReleaser
- `install.sh` — Linux/macOS: detects OS/arch, downloads tarball, verifies SHA256, installs to `/usr/local/bin` or `~/.local/bin`
- `install.ps1` — Windows: detects arch, downloads zip, verifies SHA256, installs to `%LOCALAPPDATA%\Programs\ffc`, adds to user PATH

## Architecture

```
cmd/ffc/main.go              → Entry point, calls cmd.Execute()
internal/cmd/root.go         → Root cobra command, global flags (--site, --config, --json)
internal/cmd/init.go         → init subcommand (auth method menu, --oauth/--apikey flags, writeConfig)
internal/cmd/oauth_flow.go   → OAuth PKCE flow (runOAuthInitFlow, callbackServer, PKCE helpers,
                                writeConfigOAuth, saveOAuthTokens, tryRefreshOAuthToken,
                                upsertSiteInConfig, removeSiteFromConfig, setDefaultSite,
                                buildAPIKeySiteYAML, buildOAuthSiteYAML)
internal/cmd/site.go         → site subcommand: list, add (OAuth/API key), remove, use; pickSite helper
internal/cmd/config_cmd.go   → config subcommand: TUI (no args), config get, config set;
                                escQuitKeyMap, saveConfig, updateYAMLValue, preserveHeader, resolveCfgPath
internal/cmd/ping.go         → ping subcommand
internal/cmd/get_doc.go      → get-doc subcommand
internal/cmd/list_docs.go    → list-docs subcommand
internal/cmd/create_doc.go   → create-doc subcommand
internal/cmd/update_doc.go   → update-doc subcommand
internal/cmd/delete_doc.go   → delete-doc subcommand
internal/cmd/count_docs.go   → count-docs subcommand
internal/cmd/get_schema.go   → get-schema subcommand
internal/cmd/list_doctypes.go → list-doctypes subcommand
internal/cmd/list_reports.go → list-reports subcommand
internal/cmd/run_report.go   → run-report subcommand
internal/cmd/call_method.go  → call-method subcommand
internal/cmd/update.go           → update subcommand (self-update from GitHub releases)
internal/cmd/update_check.go     → background update check; owns rootCmd.PersistentPreRunE;
                                    calls tryRefreshOAuthToken() before every non-init/update/mcp command
internal/cmd/mcp.go              → mcp subcommand (stdio/HTTP/detach mode routing, --detach, --port flags)
internal/cmd/mcp_tools.go        → 12 MCP tool definitions + handlers (registerTools, marshalResult)
internal/cmd/mcp_daemon.go       → detach logic (startDetached, runHTTPServer), status/stop subcommands, state file I/O
internal/cmd/mcp_detach_unix.go  → setSysProcAttr with Setsid=true (Linux/macOS build tag: !windows)
internal/cmd/mcp_detach_windows.go → setSysProcAttr no-op (Windows build tag)
internal/client/client.go    → FrappeClient (resty); Bearer auth for OAuth, token auth for API key
internal/client/oauth.go     → ExchangeOAuthCode, RefreshOAuthToken, GetOAuthUser (OAuthTokens struct)
internal/config/config.go    → Config/SiteConfig structs (OAuth fields), viper loading, number/date formatting
internal/output/             → Formatters: lipgloss table and JSON
internal/version/            → Build-time version variables (ldflags)
```

## Conventions

- **Error handling:** Wrap with `fmt.Errorf("context: %w", err)`. Never log and return; return and let caller decide.
- **Stdout vs stderr:** Data goes to stdout, diagnostics/errors go to stderr.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nasroykh/foxmayn_frappe_cli](https://github.com/nasroykh/foxmayn_frappe_cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
