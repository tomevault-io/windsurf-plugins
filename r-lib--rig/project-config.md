---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rig** is "The R Installation Manager" - a cross-platform CLI tool written in Rust that manages multiple R installations on macOS, Windows, and Linux. It allows users to install, remove, configure, and switch between different R versions.

## Installation Modes (user vs admin)

rig operates in one of two modes on all platforms, represented by the `Mode`
enum in `src/utils.rs`:

- **Admin mode** (the current default): R is installed system-wide and most
  operations need `sudo` / an administrator account. R goes into platform
  locations (`/opt/R` on Linux, `/Library/Frameworks/R.framework` on macOS,
  `C:\Program Files\R` on Windows) and quick links into `/usr/local/bin`
  (`C:\Program Files\R\bin` on Windows).
- **User mode**: rig installs everything into the user's home directory and
  never needs elevated privileges. R goes into `~/.local/share/rig/r`
  (`%APPDATA%\rig\data\r` on Windows) and quick links into `~/.local/bin`
  (`%USERPROFILE%\.local\bin` on Windows).

Mode resolution (`get_mode()` in `src/utils.rs`) checks, in order: the
`--user`/`--admin` global flags, the `RIG_MODE` environment variable, the
`mode` key in the rig config file, then defaults to admin. The mode is cached
after the first lookup.

Never hard-code mode-specific paths. Resolve directories through the helpers
in `src/utils.rs`, which are mode-aware and also honor override env vars /
config keys:

- `get_binary_dir()` — quick-link directory (`RIG_BINARY_DIR` / `binary-dir`).
- `get_r_install_dir()` — R installation root (`RIG_R_INSTALL_DIR` /
  `r-install-dir`).

`rig system user-mode` (`sc_system_user_mode` in `src/macos.rs`,
`src/linux.rs` and `src/windows/mod.rs`) switches an existing admin-mode setup
to user mode, reinstalls the R versions, and cleans up the admin-mode files.
The actual removal of the system-wide installations and links is delegated to
the hidden `rig system clean-admin-r` command, which self-escalates (`sudo` on
Unix, gsudo/UAC on Windows).

When editing docs or help text (`src/help/*.md`, the website under `website/`,
`README.md`), describe both modes; do not present admin-mode directories or the
`/usr/local/bin` binary location as the only behavior.

## CLI help text

Each command's help lives in one Markdown file, `src/help/<command-path>.md`
(e.g. `system-add-pak.md` for `rig system add-pak`). Within a file, the **lead
paragraph** before the first heading is the short summary (clap's `about`, shown
in `-h` and the parent command's subcommand list) and everything from the
`## Description` heading onward is the long help (`long_about`, shown in
`--help`). The dev-only `xtask` crate renders both parts to colored ANSI and
writes `ABOUT_*` / `HELP_*` `&str` constants to the committed
`src/help-generated.in`, which `src/args.rs` includes and passes to
`.about(...)` / `.long_about(...)`. Edit the Markdown, then regenerate with
`make help` (`cargo xtask gen-help`); never hand-edit `src/help-generated.in`.
CI runs `cargo xtask gen-help --check` to catch stale output. The same Markdown
files are consumed by `website/gen-cli-reference.sh` for the web reference.

You may use **Markdown links** in help files, e.g. to point at a guide page or
another command. The web CLI reference renders them as real links, while
`--help` shows only the link *text* (the xtask ANSI renderer drops the URL). So
wrapping words that are already in the prose, e.g. `[user mode](...)`, adds a
link on the website without changing the terminal output at all —
`src/help-generated.in` stays byte-for-byte identical. Because all help prose is
rendered inside `website/reference/`, write link targets relative to that
directory: `../<page>.qmd` for a guide page (e.g.
`../admin-vs-user-mode.qmd`, `../faq.qmd`, `../install.qmd`), `<cmd>.qmd` for a
sibling reference page, `#rig-<cmd>-<sub>` for an on-page anchor. (This does not
apply to clap flag/option help in `src/args.rs`, which is plain text on both the
terminal and the web.)

## Documentation website

The full user documentation is a Quarto website under `website/` (see
`website/_quarto.yml`). Prose lives in `website/_partials/*.md` (one markdown
file per section: `intro`, `features`, `known-issues`, `install`,
`macos-app`, `docker`, `faq`, `feedback`); the `.qmd` pages are thin wrappers
that `{{< include >}}` a partial. Edit the partials, not the rendered HTML.

- The site is **one level deep**: `index.qmd` (Get started — intro, quick
  start, features, known issues) plus five flat Guide pages
  (`install.qmd`, `admin-vs-user-mode.qmd`, `macos-app.qmd`, `docker.qmd`,
  `faq.qmd`), `reference/index.qmd` and `news.qmd`. Do **not** add a
  further level of sub-pages. The one exception is the **generated** CLI
  reference: `reference/` holds one page per top-level rig command
  (`reference/<cmd>.qmd`, e.g. `add.qmd`, `system.qmd`), each with its
  subcommands as sections. These pages — and the `reference/_overview.md` /
  `reference/_toc.md` includes used by `reference/index.qmd` — are produced by
  `website/gen-cli-reference.sh` (run via `make cli-reference`, which `make

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r-lib/rig](https://github.com/r-lib/rig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
