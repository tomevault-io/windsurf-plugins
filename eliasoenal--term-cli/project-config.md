---
trigger: always_on
description: This repo is **term-cli**: a single-file Python CLI that wraps **tmux** to let agents run **interactive / long-lived / blocking** terminal programs *without attaching*. It is **TTY-first** by design.
---

# AGENTS.md — Guidelines for AI Coding Agents (term-cli)

This repo is **term-cli**: a single-file Python CLI that wraps **tmux** to let agents run **interactive / long-lived / blocking** terminal programs *without attaching*. It is **TTY-first** by design.

## Read This First

**Before issuing `term-cli` commands, read `skills/term-cli/SKILL.md` end-to-end.** It’s the canonical agent playbook for correct wait strategies and common pitfalls.

- **Prefer `wait`** (prompt detection with a strong heuristic) for shells/REPLs; **use `wait-idle`** for TUIs (vim/htop/less) or screens without a detectable prompt; **use `wait-for`** only when you truly need specific text.

Do not access, write or modify files outside of the current directory without explicit permission. If you need to write temporary files, keep them in _tmp.

## Wait Semantics (Important)

- `wait` detects a **prompt** using cursor position + screen content (designed to handle SSH where old prompts remain visible).
- `run --wait` is equivalent to **send command + Enter + wait for prompt**.
- `wait-idle` is “screen stopped changing for N seconds” (best for TUIs / streaming UIs).
- `wait-for` searches the **visible screen** for **substring matches** (optionally case-insensitive). It can match echoed commands too—prefer `wait`/`run -w` when possible.

---

## CLI Cheat Sheet (Dense)

### Global

- `-L/--socket-name NAME` — use an isolated tmux server (separate socket). Useful for tests/CI.

### Sessions

```bash
# Create a sized session in a cwd, preload env, choose shell, start locked
term-cli -L t1 start -s dev -c ./proj -x 120 -y 34 -e FOO=bar -e PYTHONUNBUFFERED=1 --shell /bin/bash -l

# List (shows [LOCKED] marker)
term-cli -L t1 list

# Status (idle/running + process tree + cursor + attached + created)
term-cli -L t1 status -s dev

# Resize later (cols only here)
term-cli -L t1 resize -s dev -x 160

# Kill (refuses if humans attached unless -f; refuses if locked)
term-cli -L t1 kill -s dev -f

# Kill all (same protections; checks all unlocked first)
term-cli -L t1 kill -a -f
```

### Running + Waiting

```bash
# Run and wait for prompt to return (default wait timeout is ~10s unless -t)
term-cli run -s dev -w -t 120 "pytest -q"

# When you sent input manually (send-text -e / send-key Enter), use wait
term-cli send-text -s dev -e "python3"
term-cli wait -s dev -t 15

# TUIs / no prompt: wait until output settles
term-cli run -s dev "htop"          # no -w here
term-cli wait-idle -s dev -i 2 -t 30 # idle threshold + max timeout

# Wait for one of multiple substrings (screen match, not regex)
term-cli wait-for -s dev -t 30 -i -p "Listening on" "Ready" "Error"
term-cli wait-for -s dev -t 30 -C 3 "Error"  # print match + 3 lines of context
```

### Input

```bash
# Literal text (no Enter unless -e)
term-cli send-text -s dev ":wq" -e

# Special keys
term-cli send-key -s dev C-c
term-cli send-key -s dev Escape
term-cli send-key -s dev Up

# Mouse events in TUIs (alternate screen only)
term-cli send-mouse -s dev --text "[ OK ]"
term-cli send-mouse -s dev --text "[ OK ]" --scroll-down 3

# Multiline (stdin -> tmux buffer -> paste)
cat <<'EOF' | term-cli send-stdin -s dev
export PATH="$HOME/bin:$PATH"
echo done
EOF
```

### Capture / Logs / Scroll

```bash
# Visible screen (trimmed by default)
term-cli capture -s dev

# Last N physical rows from visible screen
term-cli capture -s dev --tail 5

# Include scrollback (-n) and ANSI escapes (-r)
term-cli capture -s dev -n 20 -r

# In alternate screen (TUI / nested tmux), --scrollback is blocked by default
# Use --force only if you explicitly want potentially stale/misleading history
term-cli capture -s dev -n 20 --force

# Annotated capture — pane metadata + highlight annotations
# Shows screen mode, bell, cursor position (row,col 1-based), mouse mode (if enabled), and highlighted TUI elements
term-cli capture -s dev -a

# Plain capture auto-enables annotations only for active alternate-screen TUIs
# Force plain output with --no-annotate
term-cli capture -s dev --no-annotate

# Optional 1-based line numbers for visible-screen captures
term-cli capture -s dev -a --line-numbers

# Pipe output to a file; strips ANSI by default (use -r for raw)
term-cli pipe-log -s dev /tmp/dev.log
term-cli unpipe  -s dev

# Scroll viewport (negative = up). Note: uses copy-mode scroll.
term-cli scroll -s dev -50
```

### File Transfer

```bash
# Upload file (gzip-compressed, SHA-256 verified; requires python3 on target)
term-cli upload -s dev local.txt remote.txt -t 30

# Download from session
term-cli download -s dev remote.txt local.txt -t 30

# Overwrite existing
term-cli upload -s dev config.json config.json -f

# Pipe support
cat data.json | term-cli upload -s dev - /app/data.json
term-cli download -s dev /app/output.csv - | head -5

# Verbose (shows compression ratio, hash verification)
term-cli upload -s dev archive.tar.gz archive.tar.gz -v
```

---

## Abbreviations (Prefix Matching)

Any **unambiguous prefix** works.

**Known ambiguous prefixes (avoid):**
- `st`, `sta` → **start** vs **status**
- `re` → **resize** vs **request***

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EliasOenal/term-cli](https://github.com/EliasOenal/term-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
