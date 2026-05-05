---
trigger: always_on
description: tmux-assistant-resurrect persists AI coding assistant sessions (Claude Code,
---

# Guidelines for AI Coding Agents

## Project overview

tmux-assistant-resurrect persists AI coding assistant sessions (Claude Code,
OpenCode, Codex CLI) across tmux restarts. It hooks into tmux-resurrect to save
session IDs and restore them automatically.

## Architecture

- `tmux-assistant-resurrect.tmux` -- TPM plugin entry point (sets tmux options, installs hooks)
- `hooks/` -- Native hooks/plugins for each assistant tool (write session IDs to state files)
- `scripts/lib-detect.sh` -- Shared library: `detect_tool()`, `pane_has_assistant()`, `posix_quote()`
- `scripts/save-assistant-sessions.sh` -- Resurrect post-save hook (process detection + session IDs + enriched fields via `extract_cli_args()`)
- `scripts/restore-assistant-sessions.sh` -- Resurrect post-restore hook (resumes assistants with CLI flags + env vars)
- `config/` -- tmux configuration snippet (used by `just install`, not TPM)
- `docs/design-principles.md` -- Detection approach, session ID extraction, process title behavior
- `justfile` -- Developer recipes (install, uninstall, status, test); end users use TPM
- `test/` -- Docker-based integration tests with real CLI binaries

## Design constraints

- **No wrapper scripts**: Do not create wrapper functions/aliases around `claude`,
  `opencode`, or `codex`. Use native hook/plugin systems instead.
- **Restore hook is the sole launcher**: Assistants must NOT be listed in
  `@resurrect-processes`. The post-restore hook handles all resuming with correct
  session IDs. Adding them to `@resurrect-processes` causes double-launch.
- **TPM-only installation for end users**: Users install via TPM (`set -g @plugin
  'timvw/tmux-assistant-resurrect'` + `prefix + I`). The `justfile` recipes are
  for developers only.
- **Pipe delimiter in tmux format output**: tmux 3.4 converts tabs and control
  characters in `-F` output. Use `|` as delimiter (documented limitation: paths
  containing `|` will break, but `|` is extremely rare in directory names).
- **Two-guard restore**: The restore script has two independent guards before
  injecting a resume command into a pane: (1) the pane's foreground process must
  be a known shell, and (2) the pane must not already have a running assistant
  in its process tree. Both must pass. This prevents typing into TUIs or
  double-launching.
- **Restore shell whitelist**: Guard 1 strips a leading `-` (login shells report
  as `-bash`, `-zsh`, etc.) then checks against a hardcoded whitelist: `bash`,
  `zsh`, `fish`, `sh`, `dash`, `ksh`, `tcsh`, `csh`, `nu`. If a user's shell
  isn't in this list, restore silently skips the pane. Update the whitelist in
  `scripts/restore-assistant-sessions.sh` if needed.

## Detection approach

Agent detection uses direct process inspection: the save script takes a single
`ps -eo pid=,ppid=,args=` snapshot and matches child processes of tmux pane
shells against known assistant binary names via `detect_tool()` in
`scripts/lib-detect.sh`.

Session ID extraction uses tool-native mechanisms (state files, process args,
JSONL lookup, SQLite database) -- this is infrastructure plumbing, not heuristic
classification. Both Claude and OpenCode overwrite their process titles, but
on macOS arm64 (v2.1.44+) process args are still visible via `ps -eo args=`.
State files and database queries remain the primary extraction methods, with
process args as a reliable fallback.

## Key conventions

- All scripts use `set -euo pipefail`
- State files go to `$TMUX_ASSISTANT_RESURRECT_DIR` (default: `$XDG_RUNTIME_DIR` or `$TMPDIR` + `/tmux-assistant-resurrect`)
- State files contain the full tool-provided context (merged from hook stdin /
  plugin events) plus plugin metadata (`tool`, `ppid`/`pid`, `timestamp`, `env`).
  The Claude hook merges Claude's entire SessionStart JSON; the OpenCode plugin
  captures the full Session object. The save script reads `session_id`, `model`,
  and `env` from state files and `cli_args` from `ps` process args. The restore
  script uses `cli_args` to reconstruct the original CLI invocation and restores
  user-configured env vars (from `@assistant-resurrect-capture-env`) as a command
  prefix.
- The `env` object in state files captures `TMUX_PANE` and `SHELL` by default,
  plus user-configured vars via `@assistant-resurrect-capture-env` tmux option
  (space-separated list, set in tmux.conf)
- Log files go to `~/.tmux/resurrect/assistant-{save,restore}.log` (truncated to 500 lines per run)
- Process inspection uses `ps -eo pid=,ppid=` (not `pgrep -P` -- unreliable on macOS)
- Agent detection matches binary names via `case` patterns in `detect_tool()`
- Hook matching in jq uses `(.command // "") | contains("claude-session-track")`
  (not exact `==`) to tolerate quoting changes across versions and ensure backward
  compatibility. The `// ""` null-coalescing prevents crashes on hook entries that
  lack a `.command` field (e.g., URL-type hooks added by other tools)
- Use `posix_quote()` from `lib-detect.sh` for any values sent to tmux panes
  via `send-keys` (safe for bash, zsh, fish, and other POSIX-ish shells)
- Hook command paths use single quotes (`bash '${CURRENT_DIR}/hooks/...'`);
  this breaks if the install path contains a single quote (unlikely with TPM)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timvw/tmux-assistant-resurrect](https://github.com/timvw/tmux-assistant-resurrect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
