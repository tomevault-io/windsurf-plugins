---
trigger: always_on
description: |
---


# /wechat-remote-control

Determine whether the user wants **login**, **attach**, or **sync**, then follow the steps below.

If the user just says `/wechat-remote-control` with no args, default to **attach**.

**Key principle:** When any step encounters a problem, fix it directly by running commands.
Never ask the user to copy-paste and run commands themselves — handle everything here.

---

## Architecture overview (for Claude's reference)

The bridge uses a **tmux-injection** model, bundled in this skill directory. It drives two
coding agents — **Claude Code** (`claude`) and **OpenAI Codex CLI** (`codex`) — auto-detecting
which one runs in each tmux pane via process ancestry. The multi-session registry records a
`kind` per session, so claude and codex sessions can coexist and be switched with `#sw`.

- **Bridge daemon** (`node src/index.js`): polls ilink WeChat API for messages, injects
  them into the user's tmux-hosted agent session via `tmux send-keys`.
- **Hook server**: listens on Unix socket `/tmp/cc_wechat_hook.sock`. Agent hooks send events
  here via `hook.py`. Claude Code fires PreToolUse / Stop / Notification; Codex fires
  PreToolUse / Stop / UserPromptSubmit (Codex has no Notification event).
- **Response forwarding**: on Stop, the bridge forwards the assistant response to the WeChat
  message it injected. For Claude it parses the transcript JSONL; for Codex it uses the Stop
  payload's `last_assistant_message`, gated on the rollout's latest user message matching what
  was injected. Terminal-initiated responses are NOT forwarded.
- **Auto-approve**: PreToolUse returns `permissionDecision: "allow"` (plus the legacy
  `decision: "approve"`). Both agents honor this — Codex honors `permissionDecision` and
  ignores `decision`, so one payload works for both.
- **All state lives in one directory**: `~/.wechat-remote-control/`
  - `accounts/<accountId>.json` — WeChat credentials
  - `state.json` — tmux target, autoApprove, transcriptPath (legacy single-session)
  - `sessions.json` — multi-session registry (active session + per-tmux-target metadata)
  - `ilink_session.json` — ilink long-poll session cache
  - `get_updates_buf` — ilink sync buffer cursor
  - `bridge.json` / `bridge.pid` / `cc_pid` — daemon metadata
  - `logs/bridge-YYYY-MM-DD.log` — rotated logs (30-day retention)
  - `history.jsonl` — injected messages and forwarded responses

**Critical: process kill safety.** Do NOT use `pgrep -f` or `grep` with bridge path strings
in the same bash command that does other work. Claude Code wraps commands in `bash -c "..."`,
so the pattern matches the shell itself, causing self-termination. Always use the Python
`/proc` scanner shown below, in a **separate** bash call from the start command.

**Environment variables this skill respects:**
- `CLAUDE_CODE_REMOTE=true` — set in cloud sessions. The bridge cannot work in cloud
  (no local tmux), so the skill refuses early with a clear message.
- `CLAUDE_CONFIG_DIR` — relocates `~/.claude/` (undocumented but supported by Claude
  Code; see anthropics/claude-code#3833). Both `detect.py` and the bridge daemon honour
  it when looking up `<config>/projects/<encoded-cwd>/*.jsonl` transcript files.
- `CODEX_HOME` — relocates `~/.codex/` (documented by Codex). Both `detect.py` and the
  bridge daemon honour it when looking up `<home>/sessions/YYYY/MM/DD/rollout-*.jsonl`
  rollout transcripts and when merging hooks into `<home>/hooks.json`.
- `CLAUDECODE=1` — set by Claude Code only. Treated as a hint, NOT a requirement: agent
  kind is determined by process ancestry, since Codex does not set it.

**Helpers:**
- `detect.py` walks `/proc` (Linux) or `ps` (macOS) up from the bash subprocess to find a
  supported agent ancestor (`claude` or `codex`), reports `agent=<kind>`, then verifies it
  lives inside a `tmux list-panes -a` pane. Used by attach Step 1 (preflight) and Step 3
  (state-file writer).

---

## login — Authenticate WeChat account

Run this once before first use, or whenever the bridge reports session expiry.

### Step 1: Check if already logged in

```bash
ls ~/.wechat-remote-control/accounts/*.json 2>/dev/null | head -1
```

If account files exist, note them and ask the user whether they want to re-login
or if this was triggered by a session expiry. If they're just setting up fresh, proceed.

### Step 2: Locate skill directory and ensure dependencies

```bash
echo "=== node/npm prereq ==="
command -v node >/dev/null && node --version || echo "NO_NODE"
command -v npm  >/dev/null && npm  --version || echo "NO_NPM"
echo "=== skill dir ==="
SKILL_DIR=$(find "$HOME" -maxdepth 7 -type f -name "login.js" 2>/dev/null \
  | grep "wechat-remote-control/dist/wechat/login.js" | head -1 \
  | sed 's|/dist/wechat/login.js||')
echo "SKILL_DIR=${SKILL_DIR:-NOT_FOUND}"
```

The QR generator is built into the skill (`dist/wechat/qrcode.js`, zero external deps),
so there is no `npm install` step for QR rendering.

**If `NO_NODE` or `NO_NPM`:** install them yourself — don't push the work onto the user.
Try the system package manager that's available, in this order, and stop on first success:

```bash
if command -v apt-get >/dev/null; then sudo -n apt-get install -y -qq nodejs npm 2>&1 | tail -3 || apt-get install -y -qq nodejs npm 2>&1 | tail -3

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MatrixA/wechat-remote-control](https://github.com/MatrixA/wechat-remote-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
