---
trigger: always_on
description: > This file provides context for AI-assisted development with Claude Code.
---

# fleet

> This file provides context for AI-assisted development with Claude Code.

TUI tool for managing multiple Claude Code sessions in parallel using tmux.

## Tech Stack
- Go 1.26+, Bubble Tea + Lipgloss, tmux, SQLite (WAL mode)

## Build
```bash
make build    # build to build/fleet
make run      # go run
make test     # go test -race
make fmt      # go fmt
make lint     # golangci-lint
make coverage # test coverage report
```

## Commit Convention
Use [conventional commits](https://www.conventionalcommits.org/). Version is auto-computed from commits:
- `fix: ...` → patch (v0.1.0 → v0.1.1)
- `feat: ...` → minor (v0.1.0 → v0.2.0)
- `feat!: ...` or `BREAKING CHANGE: ...` → major (v0.1.0 → v1.0.0)
- `chore:`, `docs:`, `refactor:`, `test:`, `style:` → patch
- Scopes are optional: `fix(hooks): ...`, `feat(ui): ...`

## Changelog
- Each PR adds a fragment file: `changelog/unreleased/<name>.md` with `type:` frontmatter (added/improved/fixed/changed/removed)
- CI checks for fragment presence; comment `/no-changelog` to skip
- At release time, fragments are merged into CHANGELOG.md and deleted

## Release
- Comment `/ship` on any issue or PR to prepare a release
- `/ship 2.0.0` to override the version
- CI opens a release PR with changelog rolled — review and merge to release
- Merging the release PR triggers GoReleaser (binaries, GitHub Release, Homebrew)
- Install: `brew install brizzai/tap/fleet` or run `bash install.sh` (requires `gh` CLI)

## Package Structure
```text
cmd/fleet/main.go      # CLI entry point
internal/tmux/tmux.go        # Tmux abstraction (create, kill, capture)
internal/tmux/pty.go         # PTY-based attach with Ctrl+Q detach
internal/session/session.go  # Session model, status detection, claude --resume
internal/session/storage.go  # SQLite persistence (sessions + claude_session_id)
internal/git/git.go          # Git operations (branch, dirty, worktree)
internal/git/repo_info.go    # RepoInfo cache + refresh logic
internal/github/pr.go        # GitHub PR info via gh CLI
internal/hooks/              # Hook-based status detection (claude_hooks, hook_watcher, status_file)
internal/workspace/provider.go     # Provider interface + GitWorktreeProvider + ShellProvider
internal/workspace/repo_config.go  # Per-repo .fleet.json loading (legacy .bc.json supported) + ResolveProvider
internal/config/config.go    # JSON config (~/.config/fleet/config.json)
internal/naming/naming.go    # Auto-name sessions via smart heuristic (filler stripping, title-case)
internal/debuglog/           # slog-based debug logging to ~/.config/fleet/debug.log
internal/diagnostics/diagnostics.go  # System diagnostics collector for bug reports
internal/ui/                 # Bubble Tea TUI (app, sidebar, preview, dialogs, styles)
internal/ui/palette.go       # Theme palette definitions (5 built-in themes)
internal/ui/settings.go      # Settings dialog (S key)
internal/ui/bugreport.go     # Bug report dialog (! key) with diagnostics, error history, action log
internal/ui/actionlog.go     # Ring buffer tracking user actions (steps to reproduce)
internal/ui/errors.go        # Ring buffer keeping error history (errors that flash and vanish)
internal/ui/keybindings.go   # Centralized keybinding definitions
internal/ui/workspace_picker.go  # Worktree dialog (base branch + new branch + existing worktrees)
internal/ui/workspace_create.go  # Create workspace sub-dialog + PendingWorkspace phantom entries
internal/ui/command_palette.go   # Command palette dialog (: / Ctrl+P) with fuzzy search
internal/chrome/protocol.go      # Command/Response types, action constants, socket path
internal/chrome/native_host.go   # Native messaging host with Unix socket bridge
internal/chrome/client.go        # TUI-side client (connects to socket, sends commands)
internal/chrome/install.go       # NMH manifest auto-install to Chrome's NativeMessagingHosts dir
chrome-extension/                # Chrome MV3 extension (service worker, manifest, icons)
```

## Conventions
- Tmux session prefix: `fleet_`
- Session ID format: `<8hex>-<unix_timestamp>`
- SQLite DB: `~/.config/fleet/state.db`
- Sessions grouped by git repo root in sidebar with tree lines (├─/└─)
- Status: Running, Waiting, Finished, Idle, Error, Starting
- Status icons: ● (running/finished), ◐ (waiting), ○ (idle/starting), ✕ (error)
- Keybindings: j/k nav, Enter attach, Space jump to next waiting/finished, a new session (instant, repo-scoped), n new session (any repo, path autocomplete), w new worktree session (base branch + new branch), d delete (Y to also destroy workspace, D to also remove repo), z undo delete (5s window), r restart, R rename, e editor, p open PR in browser, Y quick approve (waiting sessions), / filter, : or Ctrl+P command palette, S settings, ! bug report/diagnostics, ? help, q quit
- Session hotkeys (RTS-style): `Alt+0-9` (or `=` then digit) binds the selected session to a slot; re-pressing `Alt+<N>` on a session already in slot N unbinds; `==` then digit clears any slot; plain `0-9` jumps to the bound session (double-tap within 400ms also attaches); `[N]` badge in sidebar marks bound sessions; bindings persist in SQLite `slot_bindings` table (FK cascade on session delete)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brizzai/fleet](https://github.com/brizzai/fleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
