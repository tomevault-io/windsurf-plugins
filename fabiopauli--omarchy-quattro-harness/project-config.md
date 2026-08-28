---
trigger: always_on
description: `skills/omaharness/SKILL.md` documents the API. This file documents the things
---

# AGENTS.md — operating omaharness

`skills/omaharness/SKILL.md` documents the API. This file documents the things
that cost time, or a window, when you learn them by discovery. Everything here
was established against a real Omarchy session, not inferred from source.

Read this before your first mutating call.

---

## 0. You may be running inside something you can close

Before closing, killing, or force-quitting any window, find out where you live:

```bash
p=$$; while [ "$p" -ne 1 ]; do read -r ppid comm < <(ps -o ppid=,comm= -p "$p"); \
  echo "$p $comm"; p=$ppid; done
```

An agent hosted in VS Code's integrated terminal has `code` in that chain; one
in a terminal emulator has `foot`, `alacritty`, `ghostty`. Closing that window
kills the agent mid-task — the shell dies with signal 9 and the transcript ends
without an error you can read.

**Never match windows by class for a destructive operation.** `class == "foot"`
is every terminal on the desktop, including the user's, including possibly
yours. Match on the address you got back when you created the window.

---

## 1. `hyprctl dispatch` answers `ok` to calls that do the wrong thing

This is the single sharpest edge on the whole system.

Hyprland ≥ 0.56 takes selectors as **named table fields**. Passed positionally,
the selector is not an error — it is ignored, and the dispatcher runs against
**whatever is focused**:

```lua
hl.dsp.window.close("address:0x55…")      -- ok. Closes the FOCUSED window.
hl.dsp.window.close{ window = "address:0x55…" }  -- ok. Closes that window.
```

Both return `ok`. Nothing downstream can distinguish them. Verified by
experiment: with window A focused and B targeted, the positional form killed A.

Consequences to internalise:

- **Prefer the named intents.** `hyprctl.close_window(addr)`,
  `focus_window`, `focus_workspace`, `send_shortcut`,
  `move_window_to_workspace`, `move_workspace_to_monitor` encode the selector
  correctly for both grammars. `dispatch_raw` now refuses a positionally
  addressed selector, but it only recognises selector-shaped strings — it is a
  safety net, not a guarantee.
- **The stub audit checks names, not signatures.** `doctor` reporting
  `unknown_dispatchers: []` means every dispatcher *name* exists. Hyprland's
  stub declares most of them as `fun(...)`, so nothing validates argument
  shape. A correctly-named call with the wrong argument shape passes every
  check the harness makes.
- **Rehearse destructive dispatchers on throwaway windows.** Spawn two, focus
  one, target the other, and confirm the right one died. Two windows are what
  make the result unambiguous — with one window you cannot tell "honoured the
  address" from "closed the focused window".

Finding the real dispatcher names and namespaces:

```bash
sed -n '/---@class HL.DspWindowNamespace/,/^local /p' \
  /usr/share/hypr/stubs/hl.meta.lua | grep -oP '(?<=@field )\w+'
```

---

## 2. Trust ground truth over metadata

`background_safe`, `interference`, and `warnings` describe *what the harness
believes it did*. They are honest, and they are not proof of delivery.

A key call returning `backend: hyprland-sendshortcut, background_safe: True`
means the dispatch was accepted — not that the keystroke arrived.

Prove delivery with something outside the harness:

```python
# the window writes what it received; the file is the evidence
h.dispatch_raw('hl.dsp.exec_cmd("foot -T probe -- /path/reader.sh",'
               ' { workspace = "5 silent" })')
for k in ["h", "e", "l", "l", "o"]:
    desktop.key(k, app=addr)
assert Path("/tmp/typed.txt").read_text() == "hello"
```

Screenshot diffs are weaker evidence than they look. A `less` window whose file
fits on one screen shows `(END)` before and after a `G` keypress — two
different-looking captures, zero information about whether the key landed.

---

## 3. What each input route can and cannot reach

| route | reaches | background-safe | needs |
| --- | --- | --- | --- |
| `desktop.key` → `sendshortcut` | toplevel windows, by address | **yes** | chord with an XKB spelling |
| `desktop.ax.perform` | GTK/Qt apps exposing a tree | yes | PyGObject on the *right* Python |
| `desktop.type` → `wtype` | the focused window, as text | no | `wtype` (layout-safe; see below) |
| `desktop.click_screen` → `ydotool` | anything under the cursor, incl. layer shell | no | `ydotoold` + uinput access |
| Stage 2 native pointer | window surfaces directly | yes | extension built; **unported on ≥ 0.56** |

**Keys reach an unfocused window.** Verified: with a decoy focused, twelve
chords delivered to a different window's address; focus never moved, the cursor
never moved, and the target received all of them. This is the cheapest correct
route and should be the default.

**`sendshortcut` cannot press a compositor keybind.** This is the corollary
nobody expects. `desktop.key` defaults to `prefer_dispatch=True`, which hands
the chord to the *window's surface* — so the window sees it, and the
compositor's own bind never fires. Every Omarchy shortcut is a compositor bind:
`super+m` (minimize), workspace switches, the launcher. Sending them the
default way is accepted, reports `background_safe: True`, and does nothing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabiopauli/omarchy-quattro-harness](https://github.com/fabiopauli/omarchy-quattro-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
