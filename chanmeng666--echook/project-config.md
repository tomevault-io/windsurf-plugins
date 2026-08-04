---
trigger: always_on
description: > v6.4.0 · Multi-platform: Claude Code (plugin) · Cursor (native + auto-bridge) · Codex (plugin + native). Source-of-truth for every capability is `audio-hooks manifest` (live JSON, includes `pointers`, `editor_targets`, `supported_editors`). This file is orientation only.
---

# echook — AI Operator Guide

> v6.4.0 · Multi-platform: Claude Code (plugin) · Cursor (native + auto-bridge) · Codex (plugin + native). Source-of-truth for every capability is `audio-hooks manifest` (live JSON, includes `pointers`, `editor_targets`, `supported_editors`). This file is orientation only.

<critical>
1. **`audio-hooks` CLI is the only interface.** Single Python binary, JSON output, stable error codes. Never hand-edit `user_preferences.json` — use `audio-hooks set <dotted.key> <value>`.
2. **Run `audio-hooks manifest` first** for any non-trivial task. It returns the live list of subcommands, hooks, config keys, error codes, env vars, `editor_targets`, and `pointers` (paths to SKILL/README/ARCHITECTURE/etc). Anything you want to know about this project is one command away.
3. **After editing `/hooks/`, `/bin/`, `/audio/`, `/config/`, `/cursor-hooks/`, or `/codex-hooks/`, run `bash scripts/build-plugin.sh`** to sync into `/plugins/audio-hooks/`. CI runs `--check` and fails on drift.
4. **Scope guard (two tracks only).** echook does exactly two things: **(1) audio + out-of-band notification** of editor lifecycle events — telling a user *what happened* when they can't see the Claude window (sound at the desk, spoken summary when away, glanceable desktop toast / webhook when in another app), and **(2) the status line**. Anything that is neither a notification nor a status-line segment is **out of scope by design**: wellness/breathing exercises, pomodoro/timers, gamification, opening URLs, or running side-commands during a session. The `focus_flow` feature was removed in v6.0.0 for this reason. If asked to add such a feature, push back and explain it's intentionally not part of echook rather than implementing it.
5. **AI-agent-first: no human-interactive paths.** Every operation is a non-interactive `audio-hooks` subcommand (JSON in, JSON out) or a non-interactive script. There are **no** human menus, prompts, or `curl | bash` flows — the install/uninstall scripts never prompt and emit machine-readable `next_steps` for the rare step an agent can't do (e.g. `/reload-plugins`). Do not add interactive scripts, `read -p` prompts, or "run this menu" instructions, and do not tell the user to manually edit files — drive everything through the CLI. (The human-only `configure.sh` / `test-audio.sh` / `snooze.sh` / `diagnose.py` / `quick-*` scripts were removed in v6.0.0.)
</critical>

## Install commands

| Platform | Command |
|---|---|
| Claude Code | `claude plugin marketplace add ChanMeng666/echook` → `claude plugin install audio-hooks@chanmeng-audio-hooks` → **ask the user to type `/reload-plugins`** (REPL-only, no CLI equivalent — do not fake it via Bash). |
| Cursor (native) | `audio-hooks install --cursor`. Aborts with `DUPLICATE_BRIDGE` if the Claude Code plugin is already installed (Cursor 3.2.16+ auto-bridges it — double-fire). Pass `--force` only if the user accepts the trade-off; runtime guard `DUPLICATE_BRIDGE_RUNTIME_SKIP` then suppresses the native path. |
| Codex | Plugin path: `codex plugin marketplace add ChanMeng666/echook` → `codex plugin add audio-hooks@chanmeng-audio-hooks` → ask the user to reload plugins if the REPL requires it. Native hooks.json path: `audio-hooks install --codex`; only follow `next_steps` when `feature_flag_state` is `disabled`, `disabled_legacy`, or `parse_error`. The install never round-trips user TOML. |

Verify with `audio-hooks status` + `audio-hooks diagnose` + `audio-hooks test all`.

## Hook events and matcher variants (v6.4)

**37 canonical events + 30 matcher variants.** A variant is one matcher value of a matcher-scoped event — `notification` has 8 (`notification_idle_prompt`, `notification_agent_completed`, …), `stop_failure` 8, `session_start`/`session_end` 4 each, `precompact`/`postcompact`/`setup` 2 each. Since v6.4 each is independently switchable; before that they all shared their parent's single flag.

- Variant keys are ordinary booleans in `enabled_hooks`, alongside canonical names. No nesting, no schema change.
- `audio-hooks hooks list --variants` to enumerate; `hooks enable|disable|enable-only` accept variant names.
- Live truth: `audio-hooks manifest` → `variants` and `variant_gating`.

**Gating precedence** (`hook_runner.is_hook_enabled(hook, variant)`), highest first — the second rule is the one that surprises people:

1. explicit `enabled_hooks[<variant>]`
2. `enabled_hooks[<parent>] is false` — **hard kill switch for every variant under it**
3. per-variant default (`SYNTHETIC_VARIANT_DEFAULTS`)
4. explicit `enabled_hooks[<parent>] is true`
5. built-in default set: `notification`, `stop`, `permission_request`

To keep exactly one variant of a muted parent, set that variant key explicitly — rule 1 outranks rule 2. Never reach for the parent to express "all but one".


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChanMeng666/echook](https://github.com/ChanMeng666/echook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
