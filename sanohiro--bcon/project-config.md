---
trigger: always_on
description: **NEVER run these commands unless the user explicitly tells you to:**
---

# CLAUDE.md - Project Context for Claude Code

## ⛔ NEVER RELEASE WITHOUT EXPLICIT INSTRUCTION

**NEVER run these commands unless the user explicitly tells you to:**
- `git push`
- `git tag`

Commits are fine. Push/tag requires explicit instruction like "push it" or "release it".

---

## Project Overview

**bcon** - A GPU-accelerated terminal emulator running directly on Linux console (TTY) without X11/Wayland.

Think "Ghostty for the console" - bringing modern terminal features (True Color, Sixel, GPU rendering) to bare metal Linux.

## Why This Exists

- AI coding tools (Claude Code, aider, Codex) are all CLI-based
- Developers spend most time in terminal, rarely need GUI except browser
- X11/Wayland adds unnecessary overhead for terminal-only workflows
- No existing solution combines: GPU acceleration + Sixel + True Color + Japanese input on raw console

## Design Philosophy

**bcon = GPU レンダリング基盤 + 内蔵ペイン分割・タブ**

- 画面分割・タブは内蔵（tmux/zellij は Kitty graphics パススルーを壊すため）
- セッション永続化が必要な場合は tmux / screen を併用可能
- bcon が提供するのは：美しく、ヌルヌル動く、モダンなターミナル体験

```
┌─────────────────────────────────┐
│  bcon                          │  ← GPU レンダリング + ペイン/タブ
├─────────────────────────────────┤
│  DRM/KMS + OpenGL ES           │  ← ハードウェア
└─────────────────────────────────┘
```

## Technical Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        bcon                             │
├─────────────────────────────────────────────────────────┤
│  VT Parser      │ ANSI/DEC escape sequences, Sixel     │
│  Text Shaping   │ rustybuzz (HarfBuzz compatible)      │
│  Font Rendering │ fontdue (Pure Rust FreeType)         │
│  GPU Backend    │ OpenGL ES 3.0 via EGL + GBM           │
│  Display        │ DRM/KMS direct                        │
│  Input          │ evdev + xkbcommon                     │
│  IME            │ fcitx5 via D-Bus (zbus)              │
└─────────────────────────────────────────────────────────┘
```

## Implemented Features

### Rendering
- GPU rendering via OpenGL ES (DRM/KMS + EGL + GBM)
- Pixel-aligned glyph rendering for sharp text
- Full 24-bit True Color support
- Font ligature support (FiraCode, JetBrains Mono, etc.)
- Color emoji rendering (Noto Color Emoji)

### Terminal Emulation
- Sixel graphics display
- Kitty graphics protocol support
- Configurable scrollback buffer (default: 10,000 lines)
- Mouse support: selection, wheel scroll, button events (X10/SGR/URXVT)
- Touchpad support: tap-to-click, natural scroll, disable-while-typing (via libinput)
- Touchpad gestures: pinch-to-zoom font size, 3-finger swipe for tab switching
- OSC 52 clipboard integration
- Bracketed paste mode

### Input
- Full keyboard support via evdev + xkbcommon
- Japanese input via fcitx5 D-Bus integration
- Automatic IME disable for vim/emacs/etc.
- Configurable key repeat delay/rate

### Split Panes & Tabs
- Binary tree pane layout (horizontal/vertical split)
- Pane navigation, resize, zoom
- Multiple tabs with tab bar
- Mouse click to switch pane focus
- Dead pane auto-close

### UX
- Vim-like copy mode for text selection
- Incremental search in scrollback (Ctrl+Shift+F)
- Screenshot to PNG (PrintScreen or Ctrl+Shift+S)
- Runtime font size adjustment (Ctrl+Plus/Minus)
- Visual bell on bell character
- URL detection with Ctrl+Click

### Configuration
- TOML-based configuration (`~/.config/bcon/config.toml`)
- Configurable keybinds (multiple keys per action)
- Preset support: `default`, `vim`, `emacs`, `japanese`/`jp`
- Combinable presets: `--init-config=vim,jp`

## File Structure

```
src/
├── main.rs               # Entry point, event loop
├── config/
│   └── mod.rs            # Configuration, keybinds, presets
├── drm/
│   ├── mod.rs
│   ├── device.rs         # DRM device handling
│   └── display.rs        # Mode setting, page flip
├── gpu/
│   ├── mod.rs
│   ├── context.rs        # EGL/OpenGL setup
│   ├── shader.rs         # Shader compilation
│   ├── renderer.rs       # Text rendering
│   ├── emoji_renderer.rs # Color emoji rendering
│   ├── image_renderer.rs # Sixel/Kitty image rendering
│   └── ui_renderer.rs    # UI overlay rendering
├── pane/
│   ├── mod.rs            # Pane, PaneId, PaneRect types
│   ├── split_tree.rs     # Arena-based binary split tree
│   ├── tab.rs            # Tab, TabManager
│   └── layout.rs         # Tree → pixel rect calculation
├── font/
│   ├── mod.rs
│   ├── atlas.rs          # Glyph texture atlas
│   ├── shaper.rs         # Text shaping (rustybuzz)
│   └── emoji.rs          # Color emoji handling
├── terminal/
│   ├── mod.rs
│   ├── pty.rs            # PTY handling, foreground process detection
│   ├── parser.rs         # VT escape sequence parser
│   ├── grid.rs           # Character grid/cells
│   ├── sixel.rs          # Sixel decoder
│   └── kitty.rs          # Kitty graphics protocol
└── input/
    ├── mod.rs
    ├── evdev.rs          # evdev raw input
    ├── keyboard.rs       # Keyboard event processing
    └── ime.rs            # fcitx5 D-Bus integration
```

## Rendering Philosophy

**美しく、見やすく、速く。** これが bcon のレンダリングの最優先原則。

### 鮮明さ（ぼやけ防止）
- グリフの頂点座標は必ず **整数ピクセルに丸める** (`.round()`)
- 小数ピクセル座標はテクスチャの LINEAR 補間でにじみの原因になる
- `push_char`, `push_glyph`, `push_text` すべてで適用すること

### テクスチャアトラス

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanohiro/bcon](https://github.com/sanohiro/bcon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
