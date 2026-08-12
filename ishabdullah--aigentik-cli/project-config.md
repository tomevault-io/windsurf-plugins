---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Aigentik is a personal AI assistant built to run entirely on-device in Termux (Android), though the Node.js application itself isn't Android-specific and also runs on regular Linux (useful for development). It watches a Gmail inbox over IMAP IDLE, drafts/sends replies via a local LLM (Qwen served by `llama.cpp`), and is controlled entirely through natural language — either by texting a Google Voice number or emailing the monitored inbox directly. There is no cloud AI, no external API key, and no server component; the model and all data live on the phone (or dev machine). `README.md` is now a short pitch/quick-start page; full behavioral documentation lives under `docs/` (see `docs/architecture.md`, `docs/commands.md`, `docs/scheduling.md`, etc.) — read the relevant doc before making non-trivial changes, since a lot of the "why" here isn't inferrable from the code alone.

## Commands

```bash
./install.sh              # one-shot setup: system packages, llama.cpp build, model download, npm install,
                           # starter config.json — idempotent, safe to re-run, skips whatever already exists
npm test                  # full Jest suite with coverage (tests/*.test.js)
node --experimental-vm-modules node_modules/jest/bin/jest.js --config jest.config.mjs   # same, no coverage
node --experimental-vm-modules node_modules/jest/bin/jest.js --config jest.config.mjs -t "test name"   # run a single test by name
node --check <file>.js    # syntax-check a single file (no bundler/linter in this repo)
./start.sh                 # starts llama-server if needed, then `node index.js` in the background, logs to aigentik.log
./stop.sh                   # stops it
tail -f aigentik.log        # watch live logs
```

`install.sh`, `start.sh`, and `stop.sh` all use a `#!/bin/sh`-based shebang that re-execs into bash — Termux has no `/usr/bin/env` and no bash at `/bin/bash` (it lives under Termux's own sandboxed prefix), so a plain `#!/usr/bin/env bash` or `#!/bin/bash` shebang breaks there. Preserve that pattern in any new top-level shell script.

There is no build step, linter, or type checker configured — `node --check` and the Jest suite are the only automated verification available. Tests cover `email-provider.js`, `gmail.js`, `calendar.js` (IMAP lifecycle/parsing, `.ics` invite building, deterministic date-phrase parsing), and `subcontractor-form.js`/`trades.js` (lead-form field parsing, trade normalization — pure functions, no file I/O). File-backed operations on `contacts.js`, `queue.js`, and calendar booking/slot-finding are **not** covered by Jest — they read `paths.data_dir` from the real `config.json`, so exercising them through the test suite would write into the live `data/` directory. Verify changes to those manually against an isolated sandbox config, not a symlinked one — a symlinked module's relative imports (e.g. `./config.json`) resolve against the real file it points to, not the symlink's location, so a "sandboxed" test can silently write into live `data/` anyway. See "Testing against the live model" below.

## Runtime state — not what you'd assume from git

- `config.json` and everything under `data/` (including `data/profile.json`, `data/contacts.json`, `data/calendar.json`, logs) are gitignored. A fresh clone has none of this — `config.json.example` is the template, and `index.js`'s `loadProfile()` creates a fresh `data/profile.json` on first run if one doesn't exist, deliberately leaving `owner_name`/`business_name`/`business_description` unset (see "First-run onboarding" below) rather than defaulting them to any particular deployment's values.
- The live app on this machine may already be running as a background `node index.js` process with its own `llama-server` — check `ps aux` before assuming you can freely restart it; doing so sends real email (the "online" notification or, if identity is unset, an onboarding request) to the configured `admin_email`.
- The shell's `grep` is wrapped by a broken function in this environment (routes through a missing `ugrep` binary and fails with `-G: error while loading shared libraries`). Use `command grep`, `awk`, `python3`, or the `Grep`/`Explore` tooling instead of bare `grep` in Bash.

## Architecture

**Everything flows through Gmail.** There is no direct SMS send/receive on the device — an earlier version polled Termux:API directly, but that path was removed. Google Voice forwards incoming texts as email (from an address under `txt.voice.google.com`); `email-provider.js` recognizes and parses these into SMS-shaped objects, and *replying* to that forwarded email is what turns back into a real outgoing text. This means Aigentik can reply to an existing text thread but can never originate a new, unprompted text conversation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ishabdullah/Aigentik-CLI](https://github.com/Ishabdullah/Aigentik-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
