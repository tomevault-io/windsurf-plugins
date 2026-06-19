---
trigger: always_on
description: >
---


# Browser Automation with gsd-browser

## Critical Rules

1. **The daemon auto-starts on browser commands.** `daemon health` only reports state; it does not start a session. Use `daemon start` only when you want to pre-warm or verify daemon lifecycle explicitly.
2. **Always re-snapshot after page changes.** Refs are versioned (`@v1:e1`). After navigation, form submission, or dynamic content loading, old refs are stale. Run `gsd-browser snapshot` to get fresh refs.
3. **Use `--json` when parsing output.** Use text mode when reading output yourself. Use `--json` when you need to extract values programmatically (e.g., checking assertion results, parsing snapshot refs).
4. **Positional args have no flag prefix.** Commands like `click`, `type`, `hover` take positional args — do NOT add `--selector`. See exact syntax in command reference below.
5. **Use `batch` for atomic multi-step flows.** Batch reduces round trips and keeps pass/fail checks in one call. Use separate commands when you need intermediate output (e.g., snapshot to discover refs).
6. **Use `view` when the user wants to watch or direct the browser.** The live viewer is an authenticated local workbench with Control, Annotate, Record, and Sensitive modes. Keep CLI commands on the same named session.

## Core Workflow

Every browser automation follows this pattern:

1. **Navigate**: `gsd-browser navigate <url>`
2. **Snapshot**: `gsd-browser snapshot` (get versioned refs like `@v1:e1`, `@v1:e2`)
3. **Interact**: Use refs to click, fill, hover
4. **Re-snapshot**: After navigation or DOM changes, get fresh refs

```bash
gsd-browser navigate https://example.com/form
gsd-browser snapshot
# Output: @v1:e1 [input type="email"], @v1:e2 [input type="password"], @v1:e3 [button] "Submit"

gsd-browser fill-ref @v1:e1 "user@example.com"
gsd-browser fill-ref @v1:e2 "password123"
gsd-browser click-ref @v1:e3
gsd-browser wait-for --condition network_idle
gsd-browser snapshot  # REQUIRED — old refs are now stale
```

## Command Chaining

Commands can be chained with `&&` in a single shell invocation. Browser state also persists across separate invocations through the background daemon when you stay on the same session.

```bash
# Chain navigate + wait + snapshot
gsd-browser navigate https://example.com && gsd-browser wait-for --condition network_idle && gsd-browser snapshot

# Chain multiple interactions
gsd-browser fill-ref @v1:e1 "user@example.com" && gsd-browser fill-ref @v1:e2 "password123" && gsd-browser click-ref @v1:e3
```

**When to chain:** Use `&&` when you don't need intermediate output. Run commands separately when you need to parse output first (e.g., snapshot to discover refs, then interact).

---

## Command Reference

Argument syntax: `<arg>` = required positional, `[arg]` = optional positional, `--flag` = named option. Do NOT add `--` prefix to positional args.

### Navigation

```bash
gsd-browser navigate <url>                        # Navigate to a URL
gsd-browser back                                   # Go back in browser history
gsd-browser forward                                # Go forward in browser history
gsd-browser reload                                 # Reload the current page
```

### Interaction

All selectors are **positional** — do NOT use `--selector`.

```bash
# Click
gsd-browser click <selector>                       # Click by CSS selector
gsd-browser click --x 100 --y 200                  # Click by coordinates (no selector)

# Type — positional: <selector> <text>
gsd-browser type <selector> <text>                 # Atomic fill (replaces content)
gsd-browser type <selector> <text> --slowly        # Character-by-character
gsd-browser type <selector> <text> --clear-first   # Clear field before typing
gsd-browser type <selector> <text> --submit        # Press Enter after typing

# Other interaction — all positional selectors
gsd-browser press <key>                            # Press key: Enter, Escape, Tab, Meta+A
gsd-browser hover <selector>                       # Hover over element
gsd-browser scroll --direction down                # Scroll down 300px (default)
gsd-browser scroll --direction up --amount 500     # Scroll up 500px
gsd-browser select-option <selector> <option>      # Select dropdown by label or value
gsd-browser set-checked <selector> --checked       # Check checkbox/radio (omit --checked to uncheck)
gsd-browser drag <source-selector> <target-selector>  # Drag-and-drop
gsd-browser upload-file <selector> <file>...       # Set files on <input type="file">
gsd-browser set-viewport --preset mobile           # Preset: mobile, tablet, desktop, wide
gsd-browser set-viewport --width 1920 --height 1080  # Custom dimensions
```

### Snapshot & Refs

Refs are versioned (`@v1:e1`, `@v2:e3`). The version increments each snapshot. **Old refs become stale after page changes — always re-snapshot.**

```bash
# Take snapshot
gsd-browser snapshot                               # Snapshot interactive elements, assign refs
gsd-browser snapshot --selector "form"             # Scope to a CSS selector
gsd-browser snapshot --mode <mode>                 # Semantic mode (see below)
gsd-browser snapshot --limit 80                    # Increase element limit (default: 40)

# Use refs — all positional
gsd-browser get-ref <ref>                          # Get metadata for a ref

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gsd-build/gsd-browser](https://github.com/gsd-build/gsd-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
