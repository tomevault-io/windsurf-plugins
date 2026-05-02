---
trigger: always_on
description: handles toggle, daemon, open/close, submap. New dialogs are thin wrappers.
---

# smplOS Development Principles

## For AI Assistants: How to Handle Guideline Conflicts

These instructions exist to prevent regressions and enforce hard-won decisions.
**However:** if you believe a guideline is wrong, outdated, or that a better
approach exists, **do not silently comply or silently avoid the better path.**

Instead, explicitly flag it:

> "The copilot instructions say X, but I think Y would be better because Z.
> Want me to proceed with Y, or stick with the guidelines?"

Examples of when to flag:
- You know a newer API/pattern that would improve on an established one
- A guideline would cause a bug in the current context
- Two guidelines conflict with each other
- You're about to do something the instructions prohibit but the user seems to want it

**Never silently do the "wrong" thing to comply with instructions, and never
silently skip a better solution because it conflicts with instructions. Always
ask first.**

---

## For AI Assistants: Debugging UI Code Efficiently

Hard-won lessons from a 3-hour session that should have taken 30 minutes.

### Rule 1 — Verify the exact widget target before writing any code

When the user mentions a UI element by name (e.g. "the Alt+F1 menu"), do NOT
assume which widget it maps to based on the first plausible code you find.
**Always identify the GType/widget name first.**

- Ask for a screenshot, or
- Run `GTK_DEBUG=interactive nemo` / `GDK_BACKEND=x11 GTK_DEBUG=interactive`
  to open the GTK Inspector and click the widget, or
- Grep for the keybinding handler:
  ```bash
  grep -rn "GDK_KEY_F1\|<Alt>F1\|alt.*f1" src/ --include="*.c"
  ```
  Then read what that handler actually shows — before writing anything.

**Why:** "The Alt+F1 menu" could mean the app menubar (File/Edit/View/Go) OR
a custom popup. They are completely different widgets with different APIs.
Patching the wrong one wastes N build cycles with zero visible effect.

### Rule 2 — Add a one-line probe BEFORE building the real fix

When a GTK/GLib API call might silently no-op (returns void, no error code),
verify the precondition with `g_warning()` before committing to a fix:

```c
// Before the real fix:
g_warning ("toplevel type: %s  is-window: %d",
           G_OBJECT_TYPE_NAME (gtk_widget_get_toplevel (menu)),
           GTK_IS_WINDOW (gtk_widget_get_toplevel (menu)));
```

Build, run, check `journalctl -f` or stderr. If the output disproves the
assumption, iterate on the probe — NOT on increasingly elaborate "real" fixes.
One extra build cycle with a probe saves three blind fix cycles.

Common silent failure patterns in GTK:
- `gtk_widget_get_toplevel()` on a `GtkMenu` returns the menu itself (not a
  `GtkWindow`) — `GTK_IS_WINDOW()` is always `FALSE`
- `gtk_window_set_mnemonics_visible()` called before `map` is reset to FALSE
  by GTK's own map-time handler
- `g_settings_set_*()` where a dconf value exists and no `changed::` signal fires
- Signal handlers connected before the widget is realized never fire

### Rule 3 — After 3 failed attempts using framework APIs, route around them

If you've tried 3+ variations of the same GTK/GLib API and the behavior never
changes, that API is the wrong tool for the job. Stop and ask:

> "What does this framework feature actually do mechanically?
>  Can I implement just that behaviour myself in 20 lines?"

The answer is often yes and the result is simpler:

| Problem | Wrong approach (fought framework) | Right approach (routed around) |
|---|---|---|
| Underlines in popup menu | `mnemonics_visible`, `select_first`, map signal | `gtk_label_set_markup("<u>c</u>har")` — Pango renders unconditionally |
| Key activation in popup menu | GTK mnemonic scanning (needs `GtkWindow`) | `key-press-event` + `g_object_set_data` + manual `activate_item()` |

Framework APIs are designed for the common case. Programmatically spawned popups,
non-hardware-triggered events, and non-standard widget hierarchies are edge cases
where the framework's assumptions break silently.

---

## Architecture: Cross-Compositor First

smplOS supports multiple compositors (Hyprland/Wayland, DWM/X11). Every feature
must be designed with this in mind. The goal is maximum code reuse — compositors
are a thin layer on top of a shared foundation.

### Directory Structure

```
src/shared/          ← Everything here works on ALL compositors
  bin/               ← User-facing scripts (installed to /usr/local/bin/)
  eww/               ← EWW bar, launcher, theme picker, keybind help (GTK3, works on X11 + Wayland)
  configs/smplos/    ← Cross-compositor configs (bindings.conf = single source of truth)
  themes/            ← 14 themes with templates for all apps
  apps/              ← git submodule → github.com/smpl-os/smpl-apps
    Cargo.toml       ← workspace root (shared deps, renderer-femtovg)
    smpl-common/     ← shared init library for all apps
    start-menu/      ← app crates (use workspace deps via smpl-common)
    notif-center/
    settings/
    app-center/
    webapp-center/
    sync-center/
    calendar/
  installer/         ← OS installer

src/compositors/hyprland/   ← ONLY Hyprland-specific config
  hypr/                     ← hyprland.conf sources shared bindings.conf
  packages.txt              ← Wayland-specific packages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smpl-os/smplos](https://github.com/smpl-os/smplos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
