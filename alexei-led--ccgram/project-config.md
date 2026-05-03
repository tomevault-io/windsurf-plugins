---
trigger: always_on
description: ccgram (Command & Control Bot) — manage AI coding agents from Telegram via tmux. Each Telegram Forum topic is bound to one tmux window running one agent CLI instance (Claude Code, Codex, Gemini, Pi, or a plain shell).
---

# CLAUDE.md

ccgram (Command & Control Bot) — manage AI coding agents from Telegram via tmux. Each Telegram Forum topic is bound to one tmux window running one agent CLI instance (Claude Code, Codex, Gemini, Pi, or a plain shell).

Tech stack: Python, python-telegram-bot, tmux, uv.

## Common Commands

```bash
make check                            # Run all: fmt, lint, typecheck, test, integration
make fmt                              # Format code
make lint                             # Lint — MUST pass before committing
make typecheck                        # Type check — MUST be 0 errors before committing
make test                             # Unit tests (excludes integration and e2e)
make test-integration                 # Integration tests (real tmux, filesystem)
make test-e2e                         # E2E tests (real agent CLIs, ~3-4 min)
make test-all                         # All tests except e2e
./scripts/restart.sh start            # Start local dev instance in tmux ccgram:__main__ (auto-installs Claude hooks)
./scripts/restart.sh restart          # Restart local dev instance (Ctrl-C in control pane)
./scripts/restart.sh stop             # Stop local dev instance (Ctrl-\ in control pane)
./scripts/restart.sh status           # Show control pane status and logs
ccgram status                          # Show running state (no token needed)
ccgram doctor                          # Validate setup and diagnose issues
ccgram doctor --fix                    # Auto-fix issues (install hook, kill orphans)
ccgram hook --install                  # Auto-install Claude Code hooks (all supported event types)
ccgram hook --uninstall                # Remove hook from ~/.claude/settings.json
ccgram hook --status                   # Check if hook is installed
ccgram --version                       # Show version
ccgram --help                          # Show all available flags
ccgram -v                              # Run bot with verbose (DEBUG) logging
ccgram --tmux-session my-session       # Run with flag overrides
ccgram --autoclose-done 0              # Disable auto-close for done topics
ccgram --autoclose-dead 0              # Disable auto-close for dead sessions
```

Bot commands (in Telegram topics):

```
/send [pattern]   Send workspace file to Telegram (exact path, glob, or browse)
/toolbar          Show provider-specific inline action toolbar
/history          Browse paginated message history
/sessions         Active sessions dashboard
/restore          Recover a dead topic
/resume           Scan past sessions and pick one to resume
/panes            List panes with per-pane screenshot buttons
/sync             Sync window state with tmux
/upgrade          Upgrade ccgram via uv and restart
```

## Core Design Constraints

- **1 Topic = 1 Window = 1 Session** — all internal routing keyed by tmux window ID (`@0`, `@12`), not window name. Window names kept as display names. Same directory can have multiple windows.
- **Topic-only** — no backward-compat for non-topic mode. No `active_sessions`, no `/list`, no General topic routing.
- **No message truncation** at parse layer — splitting only at send layer (`split_message`, 4096 char limit).
- **Entity-based formatting** — use `safe_reply`/`safe_edit`/`safe_send` helpers which convert markdown to plain text + MessageEntity offsets (no parse errors possible, auto fallback to plain text). Internal queue/UI code calls bot API directly with its own fallback.
- **Hook-based session tracking** — Claude Code hooks (SessionStart, Notification, Stop, StopFailure, SessionEnd, SubagentStart, SubagentStop, TeammateIdle, TaskCompleted) write to `session_map.json` and `events.jsonl`; monitor polls both to detect session changes, refresh Claude task lists in Telegram, and deliver instant event notifications. Missing hooks are detected at startup with an actionable warning.
- **Shell provider chat-first design** — text sent to a shell topic goes through the LLM for NL→command generation by default; prefix with `!` to send a raw command directly. When no LLM is configured, all text is forwarded as raw commands. Two prompt modes for output isolation and exit code detection: **wrap** (default) appends a small `⌘N⌘` marker after the user's existing prompt, preserving Tide/Starship/Powerlevel10k/etc.; **replace** replaces the entire prompt with `{prefix}:N❯` (legacy, opt-in via `CCGRAM_PROMPT_MODE=replace`). Two setup paths: **Auto-setup** (explicit shell topic creation via directory browser) configures the marker immediately without asking. **Ask flow** (external window bind or runtime provider switch to shell) shows an inline keyboard [Set up] / [Skip]; Skip is respected for the session (lazy recovery won't override). On provider switch away from shell and back, a fresh offer is shown. If marker is lost mid-session (`exec bash`, profile reload), it is lazily restored on the next command send (unless user chose Skip). Marker setup is session-scoped (PS1/PROMPT override) — never modifies shell config files.
- **Message queue per user** — FIFO ordering, message merging (3800 char limit), tool_use/tool_result pairing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexei-led/ccgram](https://github.com/alexei-led/ccgram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
