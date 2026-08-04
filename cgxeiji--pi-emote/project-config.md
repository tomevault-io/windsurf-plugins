---
trigger: always_on
description: Animated pixel-art emote widget for pi TUI. Displays a reactive avatar that changes expression based on agent activity (thinking, talking, reading, writing, tool use, etc.).
---

# pi-emote

Animated pixel-art emote widget for pi TUI. Displays a reactive avatar that changes expression based on agent activity (thinking, talking, reading, writing, tool use, etc.).

## Configuration

pi-emote uses layered configuration with deep merge. Higher-priority layers override lower ones field-by-field.

### Priority (lowest → highest)

| Layer | Path | Purpose |
|-------|------|---------|
| Extension defaults | `<ext-dir>/config.json` | Shipped defaults |
| User global | `~/.pi/agent/extensions/pi-emote/config.json` | Personal preferences |
| Project local | `.pi/extensions/pi-emote/config.json` | Project-specific overrides |

### Config Fields

```json
{
  "enabled": true,
  "debug": false,
  "size": 8,
  "readingSpeed": 4,
  "hideBelow": 80,
  "holdDuration": {
    "hi": 2000,
    "success": 1200,
    "failure": 1200
  },
  "blinkInterval": [3000, 6000],
  "talkTickMs": 120,
  "cycleMs": 500,
  "emotes": [
    { "model": "*", "emote-set": "default" }
  ],
  "terminals": [
    { "match": "zellij", "render": "ascii" },
    { "match": "tmux", "render": "auto" },
    { "match": "screen", "render": "ascii" },
    { "match": "wezterm", "render": "iterm2" },
    { "match": "ghostty", "render": "kitty" }
  ]
}
```

- **enabled** — Toggle the widget on/off.
- **debug** — Enable debug logging to `debug.log` in the extension directory.
- **size** — Avatar width in terminal cells.
- **readingSpeed** — Words per second used to pace talk animation duration.
- **hideBelow** — Hide widget when terminal is narrower than this (columns).
- **holdDuration** — How long (ms) to display hi/success/failure before transitioning.
- **blinkInterval** — Random range `[min, max]` (ms) between idle blinks and think swaps.
- **talkTickMs** — Interval (ms) between mouth frame changes during talk.
- **cycleMs** — Frame cycle interval (ms) for read/write/tool animations.
- **emotes** — Model-to-emote-set mapping (see below).
- **terminals** — Terminal-to-renderer mapping (see below).

You only need to include fields you want to override. Unspecified fields inherit from lower-priority layers.

### Minimal Override Example

```json
{
  "size": 12,
  "holdDuration": { "hi": 3000 }
}
```

This changes only `size` and `holdDuration.hi`; all other settings keep their defaults.

## Emote Sets

Emote sets are directories containing frame images organized by state.

### Model-Based Selection

The `emotes` array maps model IDs to emote sets using glob patterns:

```json
{
  "emotes": [
    { "model": "*", "emote-set": "default" },
    { "model": "*opus*", "emote-set": "serious-avatar" },
    { "model": "*flash*", "emote-set": "speedy" }
  ]
}
```

- Patterns use glob syntax (`*` = any characters, `?` = single character).
- Matching is case-insensitive against the model `id` (e.g. `claude-opus-4.6`).
- **Last match wins** — order matters.
- If multiple non-catch-all patterns match, a warning is logged.
- The `emotes` array uses **append** semantics: entries from all config layers are concatenated (extension → user → project). Since last match wins, higher-priority layers naturally override lower ones. An empty array `[]` is treated as "not set" and skipped.

## Terminal Renderer Overrides

The `terminals` array maps detected terminal/multiplexer names to specific image renderers. This patches cases where pi-tui's auto-detection is incorrect.

### How It Works

1. **Multiplexer detection** (checked first): env vars like `ZELLIJ`, `TMUX`, `TERM=screen*` identify multiplexers.
2. **Terminal detection**: `TERM_PROGRAM`, `KITTY_WINDOW_ID`, `WEZTERM_PANE`, etc. identify the terminal emulator.
3. **Whitelist lookup**: the detected name is matched against the `terminals` array — first match wins.
4. **Fallback**: if no match, pi-tui's `getCapabilities().images` is used.

### Detected Names

| Name | Detected via |
|------|-------------|
| `zellij` | `$ZELLIJ_SESSION_NAME` or `$ZELLIJ` |
| `tmux` | `$TMUX` or `$TERM` starts with `tmux` |
| `screen` | `$TERM` starts with `screen` |
| `kitty` | `$KITTY_WINDOW_ID` or `$TERM_PROGRAM=kitty` |
| `ghostty` | `$GHOSTTY_RESOURCES_DIR` or `$TERM_PROGRAM=ghostty` |
| `wezterm` | `$WEZTERM_PANE` or `$TERM_PROGRAM=WezTerm` |
| `iterm2` | `$ITERM_SESSION_ID` or `$TERM_PROGRAM=iTerm.app` |
| `vscode` | `$TERM_PROGRAM=vscode` |
| `alacritty` | `$TERM_PROGRAM=alacritty` |
| `unknown` | Nothing matched |

### Render Values

- `"kitty"` — Kitty graphics protocol (direct passthrough, experimental in tmux)
- `"kitty-unicode"` — Kitty Unicode placeholders (pane-safe, experimental in tmux)
- `"iterm2"` — iTerm2 inline image protocol (experimental in tmux)
- `"ascii"` — Text-only fallback
- `"auto"` — Auto-detect: checks passthrough support and detects outer terminal

### Shipped Defaults

```json
{
  "terminals": [
    { "match": "zellij", "render": "ascii" },
    { "match": "tmux", "render": "auto" },
    { "match": "screen", "render": "ascii" },
    { "match": "wezterm", "render": "iterm2" },
    { "match": "ghostty", "render": "kitty" }
  ]
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cgxeiji/pi-emote](https://github.com/cgxeiji/pi-emote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
