---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Postcard is a GTK 4 / libadwaita email client written in Python, built and shipped as a Flatpak.

## Build & run

Everything goes through the Flatpak toolchain via [`just`](justfile) — there is **no host-level `python app.py`**. The Flatpak build compiles the **current working tree, uncommitted edits included** (the manifest uses a `"dir"` source), so you never need to commit to test a change.

```bash
just init      # one-time: add Flathub, install the GNOME 50 runtime + SDK
just build     # build the Flatpak from the working tree, install --user
just run       # build, then launch (the normal dev loop)
just run-debug # run with G_MESSAGES_DEBUG=all
just inspect   # run with the GTK Inspector open
just bundle    # produce a single-file postcard-<version>.flatpak
just check     # ruff check + ruff format --check + pyright
just lint      # flatpak-builder-lint on the manifest
```

Requires `flatpak` + `flatpak-builder` on the host; Python/GTK/meson come from the GNOME SDK. `just build` passes `--disable-updates` to reuse already-cloned sources (e.g. blueprint-compiler) — drop it if you bump a source's tag/commit in `in.gxanshu.postcard.json`.

## Lint, format, tests

- **Format:** `just fmt` (ruff, line length 88, py312 target). Editor config in `pyproject.toml`; Zed formats on save.
- **Tests:** `just test` (host pytest, no Flatpak). `build`, `run` and `bundle` all depend on it, so a failing test blocks the Flatpak build; CI runs the same suite via `.github/workflows/tests.yml`, which `release.yml` calls before publishing. The GTK-free `core/` modules are unit-tested; tests live under `tests/` mirroring the `core/` package layout (e.g. `tests/core/test_threader.py`). `[tool.pytest.ini_options]` sets `pythonpath = ["src"]`, so the package resolves without being installed — no `conftest.py` needed. Note: the checked-in `.venv` is dev-tooling only (per `pyproject.toml`) and `.venv/bin/pytest` has a stale shebang after the project rename, which is why the recipe invokes `.venv/bin/python -m pytest`. Testable without a display: the pure-Python `core/` code (threader, compose, mime parser, models, database), `mail_sync`'s folder classification and its raw-header mapping, and the IMAP/SMTP sessions via a fake in place of `imaplib`/`smtplib` (see `tests/core/net/`). Not testable: `core/secrets.py` (libsecret), the D-Bus half of `core/goa.py` (its pure host/port/security helpers are), and the whole GTK layer, since the Adw typelib only exists inside the Flatpak. That last gap is real — four crashes have shipped in `window.py` code no test could have caught, so exercise those changes with `just run`.
- `pyright` runs in `basic` mode over `src/postcard`. `PyGObject-stubs` must be installed for this to be useful — without it every `gi` import reports as unresolved and drowns the real findings. Install it with `--no-deps`: it declares `PyGObject` as a dependency, which builds `pycairo` from source and needs cairo headers, but the `.venv` deliberately takes `gi` from apt via `--system-site-packages`.
- **Check:** `just check` (ruff check + `ruff format --check` + pyright). `test` depends on `check`, and `build`/`bundle` depend on `test`, so a lint or type error blocks the Flatpak build. CI runs the same three steps.

## Coding standards

The project follows `.claude/skills/coding-standards`, enforced mechanically by `just check` — see the annotated `[tool.ruff.lint]` block in `pyproject.toml`. Rules the standard states that a linter can't check (boolean `is_`/`has_` prefixes, plural collection names, logs that name the resource) are on the reviewer.

Three places the standard is deliberately not applied literally, each with the reason recorded at the site:

- **`TRY003` is disabled.** It forbids long messages outside the exception class, which is in direct conflict with the standard's own logging rule — `ImapError(f"could not move {uid} to {destination}: {data}")` is the message we want, and satisfying `TRY003` means one exception subclass per message.
- **"Fewer than 4 parameters" cannot apply to `core/models/*.py`.** `Account`, `Email` and `Folder` are `GObject.Object` subclasses because `Gio.ListStore` only accepts GObjects, so they can't become dataclasses and every field has to be a constructor parameter. They are keyword-only instead, so call sites still read like a dataclass.
- **"Under 200 lines per file" is the target for `core/` only.** A templated widget class can't be split below its own widget surface without fighting the template, so `@Gtk.Template` modules are measured by whether the split earns its keep rather than by a line count. `window.py` is ~2,200 lines because the alternative was eight files plus a declaration-only ninth — see the main window section below.

### Logging


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gxanshu/postcard](https://github.com/gxanshu/postcard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
