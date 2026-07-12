---
trigger: always_on
description: This file guides AI coding agents (such as Claude Code and Codex) working with code in this repository. It is also exposed as `AGENTS.md` through a symlink, so both filenames resolve to the same guidance.
---

# Repository guide for AI coding agents

This file guides AI coding agents (such as Claude Code and Codex) working with code in this repository. It is also exposed as `AGENTS.md` through a symlink, so both filenames resolve to the same guidance.

## Commands

```bash
make build                  # build ./slackline binary
make test                   # go test ./... -v
make vet                    # go vet ./...
make clean                  # remove built binary
make release VERSION=1.2.3  # tag + push (requires clean working tree)

go test ./cmd/... -run TestFunctionName -v   # run a single test
go build -ldflags "-X main.version=dev" -o slackline .  # build with version
```

**Linter:** `golangci-lint run ./...` — uses `gofumpt` (stricter than `gofmt`) and `gci` for import ordering. Run before committing; the pre-commit hook also runs it.

**Hooks (lefthook):** pre-commit runs `golangci-lint` + `go vet`; pre-push runs `go test`. Do not skip hooks.

## Architecture

### Package roles

- **`main.go`** — entry point only. Declares `var version = "dev"`, calls `cmd.SetVersion(version)`, then `cmd.Execute()`. Version is injected at release via `-ldflags "-X main.version=..."`.
- **`cmd/`** — one file per Cobra subcommand. `root.go` wires the `--config` persistent flag and `loadConfig()` helper used by all runtime commands. Shared helpers (`isAuthError`, `writeMessage`) live in `ask.go`.
  - `cmd/provision.go` — `provision NAME` (per-bot, no interaction, machine-readable JSON output) and `provision bootstrap` (one-time per machine; env-var-or-stdin to seed `provision.json`). Calls `tooling.tokens.rotate` and `apps.manifest.create`.
  - `cmd/react.go` — `react add` / `react remove` subcommands. Idempotent: `already_reacted` and `no_reaction` Slack errors are silently treated as success (`no_op: true` in output).
  - `cmd/download.go` — `download --file ID --out PATH|'-'`. Fetches file metadata via `files.info` then downloads via authenticated `url_private` GET. Atomic write via `.tmp` + rename. `--out -` streams to stdout.
  - `cmd/create.go` — migration stub. Returns `errs.Usage` error with a pointer to `slackline provision`. The command is still registered (not hidden) so users get a helpful error instead of "unknown command".
  - `cmd/users.go` — `users [--match <query>]` lists/searches workspace users (`{id, handle, display_name, real_name}`, text + json), mirroring `channels.go`. `filterUsers` drops deactivated users and substring-matches ID/handle/display/real name. This is the agent's name→ID path for mentions.
  - `cmd/send.go` — `linkifyMessage` rewrites `@handle` to `<@ID>` (via `slack.UserDirectory`) before posting so mentions notify; default on, `--no-link-names` opts out. Unresolved tokens stay literal with a stderr `warning:`; an outright resolution failure (e.g. missing `users:read`) is a hard error, not a silent degrade. Resolution only contacts Slack when the body contains an `@token`. `cmd/ask.go` reuses `linkifyMessage` for the message it posts and `resolveNames` for its reply output, so it mirrors both `send` (`--no-link-names`) and `read` (`--no-resolve-names`).
  - `cmd/read.go` — when name resolution is on (default; `--no-resolve-names` opts out), `resolveNames` enriches in-text `<@ID>` to `<@ID|handle>`, sets `messageOutput.UserName`, and renders the author as `ID|handle` in text. Costs one `users.list` per read; a lookup failure warns and falls back to raw IDs.
- **`config/`** — defines `Config` and `ProvisionConfig` structs, `Load`/`Save`, `DefaultPath()`/`DefaultProvisionPath()`. Env vars `SLACKLINE_BOT_TOKEN` and `SLACKLINE_APP_TOKEN` are applied inside `Load()` after reading the file, overriding file values.
- **`errs/`** — `SlackError` type with `Code` field (`Success`/`SlackAPI`/`Auth`/`Config`/`Usage`/`Timeout` = 0–5). `WriteError` writes `{"error":"...","detail":"..."}` to stderr. `Execute()` in root maps returned errors to exit codes; cobra's own usage failures (unknown/missing flags, wrong argument counts, unknown commands) are mapped to `Usage` (4) via `configureUsageErrors`/`isCobraUsageError` in `root.go`.
- **`slack/`** — thin wrapper around `slack-go/slack`. `Client` handles `AuthTest`, `PostMessage`, `GetConversationHistory`, etc. `Resolver` provides channel name→ID resolution with in-process caching (no disk cache). `slack/users.go` adds `UserDirectory`: one cached `users.list` fetch backing forward resolution (`ResolveHandle`: `@handle`→ID, handle-first then unique display/real name), reverse lookup (`Name`: ID→handle), `LinkifyMentions` (`@handle`→`<@ID>`, regex-anchored to skip emails, trailing-dot aware), and `EnrichMentions` (`<@ID>`→`<@ID|handle>`). `slack/files.go` adds `UploadFiles` for multi-file batched upload via raw HTTP to `files.getUploadURLExternal` + `files.completeUploadExternal` (the `UploadFileV2` method does not exist in v0.19.0).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prime-radiant-inc/slackline](https://github.com/prime-radiant-inc/slackline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
