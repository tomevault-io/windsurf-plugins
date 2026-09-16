---
trigger: always_on
description: Read this before installing or modifying Beckon on a user's machine.
---

# Beckon — notes for coding agents

Read this before installing or modifying Beckon on a user's machine.

## What it is

A voice agent for Omarchy users; it also runs on other Hyprland-based setups with small adjustments. `live.py` holds a Gemini Live API
session (audio both ways over one WebSocket) and executes the tools in
`tools.py` against Hyprland. `ui.py` + `ui.html` is a local control panel on
`127.0.0.1:8777`. `tour.py` is a self-narrating demo; `narrate.py` generates
its audio with the Gemini TTS model.

Independent project built for Omarchy users — not affiliated with Omarchy, Hyprland, or Google.

## Install on Omarchy (verified steps)

1. Dependencies. Everything but the Gemini SDK is in the official repos.
   ```
   sudo pacman -S --needed python-websockets python-sounddevice wtype grim wl-clipboard ydotool libnotify python-gobject at-spi2-core
   yay -S --needed python-google-genai
   ```
   Check: `python3 -c "import websockets, sounddevice, google.genai"` prints nothing.
2. Install. As a package, so pacman owns the files and removal is clean:
   ```
   git clone https://github.com/Steven-Tibbs/beckon.git
   cd beckon/packaging && makepkg -si
   ```
   `makepkg -s` resolves dependencies through pacman, which knows nothing about
   the AUR, so `python-google-genai` must already be installed (step 1).
   Without makepkg, `cd beckon && ./install.sh` installs into `~/.local`.
   Check: `command -v beckon` resolves, and Beckon appears in the app grid.
3. API key. Never handle it yourself — tell the user to run `beckon ui` and
   paste it, or to write it themselves:
   `install -m 600 /dev/null ~/.config/beckon/api_key` then edit the file.
   Check: `wc -c < ~/.config/beckon/api_key` is roughly 39; keys start `AIza`.
4. Keybinding: `beckon setup` (or `beckon setup F7` for another key). It appends
   to `~/.config/hypr/bindings.lua`, backs it up, and refuses a key that is
   already bound. Check `hyprctl configerrors` is empty afterwards.
5. Reading full pages (optional but worth it). Both are required; neither works
   alone, and Chromium reads the state only at startup:
   ```
   gsettings set org.gnome.desktop.interface toolkit-accessibility true
   gsettings set org.gnome.desktop.a11y.applications screen-reader-enabled true
   echo '--force-renderer-accessibility' >> ~/.config/chrome-flags.conf
   ```
   Check: restart Chrome, open a page, then `python3 /usr/lib/beckon/tools.py
   --page-text chrome` prints the page's text.
6. Mouse clicks (optional). `ydotool` needs a daemon with `/dev/uinput`
   access; see the README's *Mouse clicks* section for the system unit.
   Check: `ls -l /tmp/.ydotool_socket` is owned by the user, mode `srw-------`.
7. Smoke test: press the bound key and ask *"how many windows do I have open?"*
   — it should call `list_windows` and answer aloud. With step 5 done, open an
   article and ask it to read the bottom of the page without scrolling.

## How the pieces fit

- **Tool schema** is generated from `tools.py` function signatures and
  docstrings. To add a tool: write a function, list it in `TOOLS` at the
  bottom of the file. Docstrings are what the model reads — keep them exact.
- **Shell tools** live in `~/.config/beckon/custom_tools.json` and are loaded
  by `load_custom_tools()` at import. `{arg}` placeholders are shell-quoted.
- **Hyprland calls** use the Lua dispatcher API via `hyprctl dispatch 'hl.dsp…'`.
  Verified shapes: `hl.dsp.focus({ workspace = "2" })`,
  `hl.dsp.window.move({ workspace = "2", follow = false })`,
  `hl.dsp.window.resize({ x = 50, y = 0, relative = true })`,
  `hl.dsp.workspace.move({ monitor = "r" })`,
  `hl.dsp.cursor.move({ x = 100, y = 100, absolute = true })`.
  Classic `hyprctl dispatch movewindow l` syntax does NOT work here.
- **Coordinates** for `cursor.move` are Hyprland's logical layout coordinates —
  the same ones `hyprctl clients -j` reports in `at`/`size`. Not physical pixels.
- **Mute file.** While `$XDG_RUNTIME_DIR/beckon/mute` exists, `live.py` stops
  sending mic audio and drops the model's audio output. The tour creates it so
  the model can't hear its own narration through the speakers and answer it.
- **Half-duplex gating.** `live.py` stops sending mic audio while model audio
  is queued or playing, plus a 0.4s tail (`SPEAK_TAIL`). Without this, the
  model's voice re-enters through the laptop mic and the Live API's VAD treats
  it as an interruption -- it cuts itself off mid-sentence. `BECKON_BARGE_IN=1`
  disables the gate for headphone users. Start-of-speech VAD sensitivity is
  also set LOW to ignore faint bleed.
- **Memory** is `memory.py`, backed by `~/.config/beckon/memory.json`. Rendered
  into the system prompt at session start (`memory.render()`), capped at 40
  preferences / 30 notes / 200 chars. Tools: `remember`, `note`, `recall`,
  `forget`. The prompt tells the model to ask once for a generic target
  (email, music) and then `remember()` it. Don't add unbounded memory.
- **Edge glow** is `glow.qml`, a click-through Quickshell overlay started by
  `tools._start_pulse()` during screen reads. Quickshell ships with Omarchy;
  without it the code falls back to pulsing the focused window's border.
- **Keybinds** are read live from `omarchy menu keybindings --print` each call

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Steven-Tibbs/beckon](https://github.com/Steven-Tibbs/beckon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
