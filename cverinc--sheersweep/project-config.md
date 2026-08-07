---
trigger: always_on
description: You are likely an AI coding agent (Claude Code, Codex, Antigravity, …) being asked
---

# AGENTS.md — driving sheersweep as an agent

You are likely an AI coding agent (Claude Code, Codex, Antigravity, …) being asked
to use sheersweep. This file is your front door: read it and you can wield
sheersweep correctly on the first try — without deleting something the human wanted.
(Human-facing intro is in [README.md](README.md); the full command surface is
`sheersweep --help`, which is the SSOT — trust it over guessing.)

## What sheersweep is, in one breath

sheersweep is **the Mac cleaner you can read** — a single bash script (see
`wc -l sheersweep` for the current line count; more than half is its
nine-language strings) with **no telemetry, no daemon, no
network calls, no subscription**. The default **sweep** clears *only* regenerable
junk (caches/logs/temp the OS rebuilds) across **every** account under `/Users`,
then releases local APFS snapshots so the space actually comes back. It has a
**hard-coded never-touch list** no line in the script can reach.

**macOS only.** On a non-macOS host the disk-touching modes refuse to run and exit
non-zero; only `--version` / `--help` work everywhere.

## Driving it headless (the part you'll actually use)

```bash
sheersweep --dry-run                 # preview the sweep — deletes NOTHING (always do this first)
sheersweep                           # real sweep (re-execs with sudo: one password prompt)
sheersweep uninstall <App> --dry-run # preview one app's full footprint across all accounts
sheersweep uninstall <App>           # remove it → MOVES to Trash (never rm), typed-name confirm
sheersweep uninstall <formula>       # a Homebrew formula → DELEGATED to `brew uninstall`,
                                     #   dependency-checked, exact command shown, typed confirm
sheersweep uninstall <binary>        # a hand-installed bare binary → Trash, typed confirm
sheersweep uninstall <bundle-id> --dry-run   # headless preview of a REMOVED app's leftover
                                     #   data (refused if anything installed claims the id)
sheersweep tools                     # read-only CLI inventory (formulae/orphans/binaries/
                                     #   toolchain data) — changes nothing, no sudo needed
sheersweep leftovers --dry-run       # preview orphaned LaunchAgents/Daemons
sheersweep leftovers                 # remove dead/orphan startup items → Trash, yes/no confirm
sheersweep reclaim --dry-run         # preview build output — PROVEN tier eligible ONLY with 3
                                     #   proofs: gitignored + known pattern + manifest. ALSO
                                     #   previews an "unidentified" tier (below).
sheersweep reclaim                   # PROVEN: pick rows, typed count → Trash + receipt (records
                                     #   the rebuild command per folder). Then UNIDENTIFIED, one at
                                     #   a time: a heavy (≥512 MB) gitignored folder it CANNOT prove
                                     #   regenerable — never auto-picked, removed only if you type
                                     #   its name (uninstall-style). Same Trash + restore.
sheersweep reclaim --stale 30d       # only repos untouched for 30+ days (both tiers). Unidentified
                                     #   floor: SHEERSWEEP_SUSPECT_MIN_MB (default 512).
sheersweep restore                   # undo the last uninstall/leftovers/reclaim — put it all
                                     #   back (a reclaim receipt also prints rebuild commands)
sheersweep restore --list            # show past removals
sheersweep --version                 # prints the current version; never needs sudo
```

Locale is auto-detected; force it with `SHEERSWEEP_LANG=en-US|ja-JP|zh-TW|zh-Hans|ko-KR|es-ES|de-DE|fr-FR|pt-BR`.

## Non-negotiable rules (a cleaner is destructive-adjacent — break one and you hurt the human)

1. **Dry-run first, every time.** Run `--dry-run`, read what it would free, *then*
   decide. The preview deletes nothing.
2. **Never bypass the human's confirmation gate.** `uninstall` waits for the human
   to **type the app's name** (a multi-select in the picker asks instead for the
   **count of apps**, once, over one combined preview); `leftovers` / `restore`
   wait for an interactive
   yes/no. These prompts read from a TTY and are for the *human*, not you — do not
   pipe answers in, fake a TTY, or auto-confirm to "save a step." User excitement
   is not authorization.
3. **The sweep only clears regenerable junk** (`Library/Caches`, `Library/Logs`,
   `~/.cache`, `~/.npm`, Xcode DerivedData/DeviceSupport, CoreSimulator/Cargo/Gradle
   caches; system-wide `/Library/Caches`, `/.adobeTemp`, `brew cleanup`). It names
   no vendor and will never grow an "aggressive"/"deep" mode. Don't ask it to
   delete app data — that's what `uninstall` is for, on purpose (removed-app
   residue included: that's `uninstall`'s discovery section, not a sweep job).
4. **Honor the `[ HELD ]` never-touch list — and never route around it.** No line can reach:
   Photos / Documents / Desktop / Movies / Music, Clip Studio (CELSYS), app
   Containers & Application Support, Dropbox / cloud-sync folders, screen
   recordings, Mail / Messages / Keychains, any git repo, any Obsidian vault. If

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CVERInc/sheersweep](https://github.com/CVERInc/sheersweep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
