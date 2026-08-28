---
trigger: always_on
description: Context and hard invariants for Claude Code sessions in this repository.
---

# CLAUDE.md — McGrammar

Context and hard invariants for Claude Code sessions in this repository.

## What this is

A macOS menu bar utility (Swift + AppKit, SPM, **zero third-party dependencies**) that corrects the
user's selected text in any app by shelling out to their locally installed, locally authenticated
Claude Code CLI. Two independent trigger paths: a global hotkey (⌃⌥G) and an NSServices menu item.

## Invariants — do not violate these

### Credentials
- **Never** add an API key path, token handling, or any login flow. The app invokes the official
  `claude` binary the user installed and authenticated themselves. That is the entire compliance
  position; an API-key mode is a roadmap item to be revisited only against current Anthropic policy.
- `ClaudeRunner.childEnvironment()` strips `ANTHROPIC_API_KEY` and `ANTHROPIC_AUTH_TOKEN`. Keep it
  that way: a key in the environment takes precedence over the subscription login in the CLI.
- Never use `--bare` mode (API-key only).

### Sync/async structure (`ClaudeRunner`)
- `fixSync` is the core and blocks its calling thread. `fixAsync` is a thin wrapper for the hotkey
  path: background queue in, main-thread completion out.
- **The NSServices handler MUST call `fixSync` directly.** Never wrap `fixAsync` in a semaphore
  there — the handler runs on the main thread and the completion dispatches back to main, which
  deadlocks with certainty. This mistake has already been made once on this project.

### Binary discovery
- Resolve `claude` through a **zsh login shell** (`/bin/zsh -l -c 'command -v claude'`) and cache it.
  GUI-launched apps do not inherit the terminal PATH — this is the #1 silent failure mode.
- Keep the disk fallback list (`~/.local/bin`, `~/.claude/local`, `/opt/homebrew/bin`,
  `/usr/local/bin`) and keep prepending those to the child PATH.
- The resolved path (or the not-found warning) must stay visible in the menu bar dropdown.

### Invocation
- `claude -p "<PROMPT>" --max-turns 1`, with the user's text piped over **stdin** — never
  interpolated into the argument list or a shell string.
- 60s watchdog; terminate the process if exceeded.
- Keep the prompt strict. If preamble ever leaks into the output, switch to `--output-format json`
  and read the `result` field rather than tightening the prompt further.
- Drain stdout and stderr concurrently; a blocked pipe buffer wedges the child.

### NSServices (Info.plist)
- `NSMessage` must exactly equal the `@objc` selector name on `NSApp.servicesProvider`:
  `fixGrammar` ↔ `fixGrammar(_:userData:error:)`.
- `NSSendTypes` **and** `NSReturnTypes` both `NSStringPboardType`. Removing `NSReturnTypes` makes
  the service send-only and selection replacement silently stops working.
- `NSTimeout` = `120000` ms. The default is far too short for Claude Code spin-up.
- Register at launch: `NSApp.servicesProvider = provider; NSUpdateDynamicServices()`.
- macOS caches the Services menu aggressively — `make-app.sh` runs `pbs -flush`/`-update`, and the
  README documents the manual steps. Do not chase this as a bug.

### Bundle
- `LSUIElement = true` plus `NSApp.setActivationPolicy(.accessory)` — menu bar only, no Dock icon.
- Ad-hoc `codesign --force --sign -` **plus an explicit designated requirement**:
  `-r='designated => identifier "com.zernonia.mcgrammar"'`. This is what makes Accessibility (TCC)
  grants survive rebuilds. `--identifier` alone does NOT: it sets the bundle ID in the code
  directory, while codesign still derives a DR that pins the exact `cdhash`. Every rebuild then
  changes the hash and silently voids the grant, and the symptom is nasty — System Settings keeps
  showing a ticked McGrammar entry that no longer matches the binary, so the app re-prompts while
  the user is looking at a checkbox that says it is already allowed. Verify after any change to the
  signing step with `codesign -d --requirements - <app>`; it must print the identifier form, not a
  `cdhash H"..."`. Kill any running instance before replacing the bundle.
- Accessibility permission attaches to the *launching* process — test the hotkey from the .app,
  never from a terminal-launched binary.

### Privacy and cleanup
- Never log, cache, or persist user text anywhere. The README states this as a guarantee.
- The CLI itself persists what the app does not: `claude -p` writes a session transcript containing
  the corrected text under `~/.claude/projects/<cwd slug>/`. The child therefore runs in
  `~/Library/Application Support/McGrammar/cli-workspace` so those transcripts land in a project
  folder only McGrammar causes to exist, and `Transcripts.purge` deletes them after every fix.
  Keep that purge narrow — `.jsonl` only, marker-matched folder only — so an unknown CLI layout
  degrades to a no-op instead of deleting someone's history. Do **not** re-add a modification-time
  filter: it sounds safer and leaks. A transcript that misses its own run's purge (flushed late,
  delete failed, app quit mid-fix) is then older than every later cutoff and survives for good.
  The folder is ours by construction, so sweeping all of it is both safe and the point.
- `prepareWorkspace()` returning nil must fail the fix (`.workspaceUnavailable`), never fall back

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zernonia/mc-grammar](https://github.com/zernonia/mc-grammar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
