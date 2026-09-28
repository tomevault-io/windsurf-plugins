---
trigger: always_on
description: > Identity and global rules: `~/.claude/CLAUDE.md` (Larry, 9 Laws, Constitution).
---

# Showoff Omarchy — project brief

> Identity and global rules: `~/.claude/CLAUDE.md` (Larry, 9 Laws, Constitution).
> Machine context: `~/Projects/CLAUDE.md` (vic, RTX 4050, Ollama).
> This file is the project brief. Born 2026-09-21 from Fred: *"create a plugin
> that people use on their real system that is an auto showoff Omarchy that
> will run on any omarchy system … The goal isn't to show an omarchy person
> omarchy, it's to show a windows or mac person."* Re-scoped the same night
> from plugin to **application** (below).

**Status: the whole auto show is built (2026-09-26).** All 21 acts in
`app/acts.js` run in `AUTO_ORDER` on a stage workspace (the first empty one),
with live keycaps looked up from the machine's own Hyprland binds, a
reading-time pacing model, the theme and wallpaper pickers, hand-off acts,
the QR end card (keep the theme? Y/N) and restore on every exit path. There is no menu (removed at Fred's word); install.sh / PKGBUILD (P10) not written.

Verified on vic 2026-09-26 (contact sheet of a 122 s run, then drills):
takeover logo → browser → tiling → workspace flyby → theme picker driven with
Right Right Enter → roulette recolouring everything → wallpapers → wallpaper
picker → btop → aquarium → fastfetch → YouTube web app → emoji → clipboard;
theme, wallpaper and window count restored. That run was ended by Esc Esc
(Fred, watching) before menu-tour; acts 14–21 have **not** been seen on screen
yet. Fixed on the way: launches were tied to the runner watchdog and died at
15 s (btop) — every launch is now `setsid -f` detached; keycaps flashed for
0.9 s — now `readMs`/`keycapMs` pacing and keycaps stay up during the action.

### Verified on vic, 2026-09-21 (Fred's go; three runs, screenshots pixel-checked)

| Check | Result |
|---|---|
| Loads as its own process (`quickshell -n -p app/`) | yes, "Configuration Loaded", no QML errors |
| Fullscreen layer above everything | Hyprland lists `showoff-omarchy` 1920×1080 at Overlay level; scrim leaves the desktop readable |
| Theme colours from `colors.toml` | caption rendered in Phosphor's accent `#44E8CB` |
| Countdown → auto (silence) | flipped to the AUTO SHOW placeholder by itself |
| Countdown → menu (any key) | Space during the countdown → PICK AN ACT placeholder |
| Single Esc does **not** quit | sub-caption "press Esc again to stop", process alive, layer up |
| Esc×2 quits | process gone, layer gone, within 0.5 s |
| Multi-monitor | **untested** — vic had one screen (eDP-1) |
| Log noise | one harmless `qt.qpa.services` WARN: portal app-id already registered (second Qt app in the session) |
| Title sizing | at `height/8` "SHOWOFF OMARCHY" nearly spans 1920 px — needs fit-to-width before smaller screens |

**Testing rule, learned the hard way that night:** never inject a key
(`wtype`) unless `hyprctl layers -j` shows `showoff-omarchy` *immediately*
before that key. Fred pressed Esc×2 himself mid-test; my two injected
Escapes then went to the focused terminal and cancelled an AskUserQuestion
prompt in one of his other Claude sessions. Guard every key; "gone after my
keys" proves nothing on its own.

Repo: `github.com/nixfred/showoff.omarchy` (PUBLIC — it is for other
people's machines; nothing private ever goes in here).

---

## Law 0 — runs on anyone's Omarchy, depends on no other plugin

Fred, 2026-09-21: *"It must work on anyone's omarchy and not depend on other
plugins."* This outranks every other choice in the project.

- **Only stock surface.** `omarchy-*` commands in `$OMARCHY_PATH/bin`,
  Quickshell (0.3.x, present on every Omarchy 4.x because the shell *is*
  Quickshell), Qt, and stock state under `~/.local/state/omarchy/current/`
  (`theme.name`, `theme/colors.toml`, `background`). Nothing from
  `~/.config/omarchy/plugins/*`, nothing from Fred's forks or vic patches, no
  `nixfred.*` IPC targets, and **not** the shell's `qs.Commons` (that is
  only importable from inside the shell process).
- **Optional packages are preconditions, not dependencies.** btop, cava,
  cmatrix, fastfetch, qrencode, ttfx are pacman packages the show can offer to
  install *on screen* (One-Line Install) or that `showoff prepare` installs
  once. An act whose package is missing is skipped, never fatal.
- **vic is NOT a stock box.** 80+ plugins, a patched shell, Infomarchy owning
  the `background` IPC target, a dev checkout at `$OMARCHY_PATH`
  (`4.0.0.alpha`). Something working on vic proves nothing. Verify on a
  **clean profile**: Fred chose a **fresh user on `ovm`** (packaged 4.0.4-1,
  Quickshell 0.3.1; its `pi` user's plugin dir was seeded from vic, so make a
  new user — `demo` — and test there).
- **Minimum Omarchy version = 4.0** (the release that made the shell a
  Quickshell process). Say so in the README; don't chase older releases.
- When a stock command doesn't exist on a release, the act says so on screen
  and moves on. Never paper over it with a copied script.

## It is an application, not a plugin (Fred, 2026-09-21)

Fred: *"I don't think this should be a plugin. I think it should be an
application because it needs to take over a full screen and it needs to allow
the user to pick auto … or … select on a menu … maybe even as the thing
starts, it does a countdown, and if you don't pick something or move

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nixfred/showoff.omarchy](https://github.com/nixfred/showoff.omarchy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
