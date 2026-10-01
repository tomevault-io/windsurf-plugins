---
trigger: always_on
description: Instructions for an AI agent working in this repository. Read this before
---

# CLAUDE.md — working on aura-glass

Instructions for an AI agent working in this repository. Read this before
touching anything. `README.md` is the *user* document; this is the contributor
one. Deeper detail lives in [docs/](docs/) — start at [docs/README.md](docs/README.md).

---

## What this project is

A GNOME desktop theme installed by a Bash script. There is no build system, no
package, no test runner in the usual sense. `./install.sh` fetches pinned
upstreams, copies stylesheets into `~/.config/aura-glass`, splices them into the
theme's generated CSS, loads a dconf preset, and writes gsettings. Everything
lands under `$HOME` except three optional root steps (dependencies, the
`gnome-rounded-blur` library, the GDM login screen).

Targets GNOME Shell 48 / 49 / 50, Wayland preferred. Arch/CachyOS, Fedora,
Ubuntu/Debian.

---

## The five rules that matter most

### 1. One place resolves a flag into a file

The precedence chain is **typed flag → `--glass-mode` → per-mode memo →
top-level `$CONF_DIR` memo → default**, and it is implemented exactly once, in
`install.sh`'s resolution block plus the `apply_*` functions in
[lib/steps-dconf.sh](lib/steps-dconf.sh) and [lib/steps-css.sh](lib/steps-css.sh).

The settings window, the setup wizard and `aura-glass-preview` all call *that*
code rather than reimplementing it. Never add a second copy of a resolution
rule. If a GUI needs to know a value before `install.sh` runs (radius bounds,
preset rows), the duplication gets a checker — see rule 3.

### 2. Never edit `~/.themes` or `~/.config/gtk-*` by hand

Those four files are generated. `bin/aura-glass-apply` owns a single marked
block in each:

```
/* >>> aura-glass BEGIN <<< */ … /* >>> aura-glass END <<< */
```

Edit `css/*.css` in the repo, then re-run the installer. Anything written
outside that block is lost on the next apply and breaks uninstall.

### 3. Duplicated numbers are *checked*, never trusted

Radii and blur sigmas appear literally in stylesheets, in `dconf/core.ini`, and
in `gui/aura_glass_settings.py`. GTK4/St/GTK3 have no shared variable mechanism,
so the duplication is unavoidable. [tokens/tokens.sh](tokens/tokens.sh) is the
single source of truth and `tools/check-tokens.sh` fails the commit when a copy
drifts. **Change a value in `tokens/tokens.sh`, then in every consumer its
comment names, then run the checker.**

### 4. Comments are the deliverable

Almost every non-obvious number in `css/`, `dconf/` and `lib/` carries a comment
saying *why it is that number* — usually the result of a bisection against a
screenshot. Those comments are the most valuable thing in the repository. Do not
strip them, do not shorten them, and when you change a value, update the reason
too. If you cannot say why a number is what it is, do not change it.

### 5. Apply theme edits to the live desktop

A repo edit is not a finished change. After editing `css/`, `dconf/` or
`tokens/`:

```bash
./install.sh --settings-only -y
```

That reapplies the dconf preset, the CSS and the gsettings without touching the
theme or extensions. No root, no network, a few seconds. GTK apps need a
restart to pick up the GTK side; the shell reloads itself.

---

## Repository map

| Path | What it is |
|---|---|
| `install.sh` | Entry point: flag parsing, precedence resolution, step orchestration |
| `uninstall.sh` | Restoration, in four scopes (base / `--extensions` / `--assets` / `--gdm`) |
| `lib/` | The steps, split by concern. Definitions only — nothing runs at source time |
| `tokens/tokens.sh` | Single source of truth for every value written down twice |
| `css/` | Stylesheets. Numeric prefix **is** the cascade order |
| `dconf/` | `core.ini` (every extension's settings), `extras.ini`, `solid.ini` |
| `patches/` | Pinned-upstream patches (GNOME 50 compat, Blur My Shell behaviour) |
| `bin/` | User-facing commands installed into `~/.local/bin` |
| `gui/` | GTK4/libadwaita settings window and setup wizard (optional) |
| `extensions/` | The one first-party GNOME extension (`aura-glass-blur@aura-glass.local`) |
| `systemd/` | Four user units: icon sync, panel blur fix, GDM sync, update check |
| `tools/` | Checkers, the headless preview harness, the git hooks |
| `docs/` | Contributor documentation (this file's long form) |

`lib/` breakdown:

```
steps.sh              upstream pins, paths, extension arrays, preflight, install_theme, finish
steps-migrate.sh      moving a pre-rename (tahoe-glass) install onto current names
steps-extensions.sh   EGO downloads, the three pinned builds, gnome-rounded-blur
steps-assets.sh       icon and cursor packs
steps-fonts.sh        --font: Inter, MiSans, SF Pro
steps-modes.sh        the three glass modes and their per-mode memo drawers
steps-css.sh          stylesheet install, tint/opacity/radius rewriters, density
steps-dconf.sh        the dconf preset and every apply_* that writes a key
steps-integration.sh  icon-sync agent, Flatpak override, panel blur unit
steps-gdm.sh          login screen theme and monitor sync
steps-gui.sh          settings window, update-check timer
steps-wizard.sh       runs the GTK setup wizard, reads its flags back
common.sh             output helpers, run/dry-run, backups, pinned fetchers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevWebeloper/aura-glass](https://github.com/DevWebeloper/aura-glass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
