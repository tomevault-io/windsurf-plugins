---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build

GPaste uses Meson + Ninja:

```sh
mkdir build && cd build
meson ..
ninja
```

Common build options (`meson .. -Doption=value`):

| Option | Default | Description |
|---|---|---|
| `gnome-shell` | true | Build the GNOME Shell extension |
| `introspection` | true | Generate GIR data |
| `vapi` | true | Generate Vala bindings (requires introspection) |
| `systemd` | true | systemd user unit |

For a lighter build that skips the GNOME Shell extension, GIR introspection data
and Vala bindings (the daemon, UI and preferences apps are always built):

```sh
meson .. -Dgnome-shell=false -Dintrospection=false -Dvapi=false
```

Run tests from the build directory:

```sh
ninja test          # or: meson test -C build
```

Tests live under `tests/`. `tests/history/` unit-tests the `GPasteHistory` model
(add/dedup/size-enforcement/remove/select) against an in-memory `GSettings`
(`GSETTINGS_BACKEND=memory` + the schema compiled into the build tree) and a
throwaway `XDG_DATA_HOME`, so they need no display server or dconf. The `eslint`
test lints the GNOME Shell extension JS.

Check header include ordering:

```sh
tools/check-includes.sh
```

## Code style

- C standard: GNU17
- Formatting: ClangFormat (see `.clang-format`). Key rules: Allman braces, 4-space indent, no column limit, space before parens, no tabs.
- clang-format is not yet enforced; do not run it automatically.
- **Braces**: Remove braces from `if`/`else if`/`else` branches whose body is a single statement on a single line. Keep braces when the body has multiple statements OR spans multiple lines (e.g. a nested if-else chain). Multi-statement macros that need to appear as a single statement must use the `do { ... } while (0)` idiom — `SWITCH_STATE` in `gpaste-file-backend.c` does this and can safely appear without surrounding braces.

### JavaScript (GNOME Shell extension)

The `src/gnome-shell/` extension follows upstream GNOME Shell's JS conventions, enforced by the **same tooling, layout, and configuration** as upstream:

- The npm project lives in `tools/` (`tools/package.json`, `tools/package-lock.json`, `tools/eslint.config.js`), mirroring gnome-shell. The repo-root `eslint.config.js` is a **symlink** to `tools/eslint.config.js`.
- **ESLint** with [`eslint-config-gnome`](https://gitlab.gnome.org/World/javascript/eslint-config-gnome) (`recommended` + `jsdoc` configs) and the [`ci-run-eslint`](https://gitlab.gnome.org/World/javascript/ci-run-eslint) runner, both pinned to the same commits upstream uses. The config mirrors upstream's custom rule overrides (`camelcase`, `consistent-return`, `eqeqeq: smart`, `key-spacing`, `prefer-arrow-callback`, `prefer-const`, jsdoc tweaks). Shell-extension globals (`global`, `_`, `C_`, `N_`, `ngettext`) are declared for `src/gnome-shell/**`.
- Style basics live in `src/gnome-shell/.editorconfig` (LF, UTF-8, trim trailing whitespace, 4-space indent for `*.js`).
- Run it with `tools/run-eslint.sh` — exactly the upstream wrapper. It `npm clean-install`s into `tools/` on first run, symlinks `node_modules` into the repo root for import resolution, then lints `src/gnome-shell`. Pass `--fix` to auto-fix formatting.
- The same script is the single entry point everywhere: the meson `eslint` test (`meson test -C build eslint`, skipped when `npm` is absent) and the GitHub Actions workflow (`.github/workflows/eslint.yml`, runs on pushes/PRs touching the JS or tooling) both invoke it. Upstream runs lint from GitLab CI; GPaste runs it from GitHub Actions, but the toolchain, config, layout, and `run-eslint.sh` are otherwise identical.
- This tooling applies **only** to the JavaScript code; it does not affect the C/meson sources.

Code conventions (also following upstream):
- **Don't version-pin core `gi://` imports** — write `gi://GObject`, `gi://GLib`, `gi://Gio`, `gi://Pango`, `gi://Clutter`, `gi://St`. Only pin typelibs that genuinely ship multiple versions: `gi://GPaste?version=2`, `gi://GPasteGtk?version=4`.
- **Manage signal lifecycles with `connectObject`/`disconnectObject`** (owner = `this`) for connections to long-lived non-actor GObjects (settings, the `GPaste.Client`), rather than tracking handler ids and disconnecting them by hand. They auto-disconnect when the owner actor is destroyed.
- **Use a standard `constructor()` (calling `super(...)`) in `GObject.registerClass` classes**, not `_init()`/`super._init()`. GJS bridges to the `_init()`-based shell/St/Clutter base classes transparently (positional args like `super(0.0, 'GPaste')` and property dicts like `super({...})` both work).

Async / `Gio._promisify` conventions (these bit us — keep them):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Keruspe/GPaste](https://github.com/Keruspe/GPaste) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
