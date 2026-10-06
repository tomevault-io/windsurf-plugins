---
trigger: always_on
description: Guidance for anyone changing this repository, whether a person or an AI coding agent. OhMyDebn turns a Debian-based distro into a Cinnamon desktop for power users. How to propose a change (discuss first, then a PR to the `dev` branch) is in [CONTRIBUTING.md](CONTRIBUTING.md).
---

# AGENTS.md

Guidance for anyone changing this repository, whether a person or an AI coding agent. OhMyDebn turns a Debian-based distro into a Cinnamon desktop for power users. How to propose a change (discuss first, then a PR to the `dev` branch) is in [CONTRIBUTING.md](CONTRIBUTING.md).

This file is for developing OhMyDebn. The skill in `config/ohmydebn-skill/` is different: it ships to users so the AI tools on their desktop understand an installed OhMyDebn.

## Repository layout

- `bin/` - every command, named `ohmydebn-*`. Installed to `/usr/share/ohmydebn/bin`, and scripts call each other by that absolute path.
- `install.sh` - the bootstrap: checks the distro, sets up apt sources and the OhMyDebn repo, installs the `ohmydebn` package, then runs `ohmydebn.sh`.
- `ohmydebn.sh` - sources `install/{packaging,config,cleanup,finalization}/all.sh`. A new file under `install/` must be sourced from its directory's `all.sh`.
- `config/` - default configs, `*.tpl` templates rendered per theme, `tile-rules.json`, and the user-facing agent skill.
- `themes/` - the built-in OhMyDebn theme, shipped as the separate `ohmydebn-themes` package. The Omarchy themes are packaged from the omarchy repo by `build-package-ohmydebn-themes-omarchy.sh` in `ohmydebn-package-build`.
- `tests/` - the test suite (see below).
- `VERSION` - the `ohmydebn` package version.

Two sibling repositories go with this one:

- `ohmydebn-package-build` - builds every OhMyDebn deb (including `ohmydebn` itself from this repo) and publishes the apt repos. Third-party apps packaged for OhMyDebn (Codex, Pi, Grok Build, T3 Code...) get their build scripts there.
- `ohmydebn-docs` - the user documentation site and release notes.

## The install runs again on every update

`ohmydebn-update` updates the `ohmydebn` package and reruns `install.sh`, so everything under `install/` runs on new installs and on every update of existing ones. Keep it idempotent. A change that must happen only once is guarded by a marker file in `~/.local/state/ohmydebn-config/<name>`. Marker names must be unique (the consistency tests check this). Never overwrite a file the user may have edited without such a guard and a reason.

## Testing

Run the whole suite with `tests/run.sh` (`--skip-apt-checks` skips the read-only apt queries). Everything must pass before a change is done.

- `tests/lint.sh` - `bash -n` and `shellcheck -S warning` on every shell script, `py_compile` on the Python ones.
- `tests/consistency.sh` - about 40 cross-file checks: menu case arms match their items, the menu search index matches the menu tree, no duplicate packages or keybindings, every third-party apt repo is pinned, every `-install` script runs `apt update` first, every AI installer asks the default question with its own name, tile rules match launcher titles, and more. Read the failing check's comment; it explains the bug it guards against.
- `tests/unit/test-*.sh` - mocked unit tests built on `tests/lib/test-helpers.sh` (`mock_init`, `mock_bin`, `mock_log`, `assert_eq`, `assert_contains`...).
- `tests/apt-checks.sh` - read-only checks that every package OhMyDebn names exists.

Unit test rules:

- Nothing may touch the real system. Test a script by copying it with `sed`, pointing `/usr/share/ohmydebn/bin` (and any other absolute path) at the mock directory, and mocking `sudo`, `apt`, `dpkg` and friends. `install.sh` and some scripts read `OHMYDEBN_TEST_*` overrides (`OS_RELEASE`, `APT_DIR`, `SKIP_CONFIG`, `SYSTEMD_DIR`...) for paths that can't be patched.
- Always give the script under test its stdin: `</dev/null`, or the answers it should read. Inherited stdin can hang the suite.
- New behavior gets a test. A bug fix gets a test that fails without the fix; check that it does.
- Never run `install.sh`, `ohmydebn-update` or an installer on your own machine to try a change. Test real runs in a VM.

## Script conventions

- `#!/bin/bash` and `set -euo pipefail` for new scripts, two-space indentation.
- Comments explain why, and this codebase is generous with them: follow the surrounding density, and keep a comment true when you change the code under it.
- Section headings in terminal output come from `ohmydebn-headline`.
- Naming, for an app `<app>`: `ohmydebn-<app>` (opens it, installing first if needed), `-install`, `-remove`, `-run` (the payload inside a terminal window), `-cli` (runs in the current terminal), `-gui`, `-tiled`.
- Installers:
  - check `dpkg -s` first, and do nothing if the package is already installed
  - ask "Press Enter to continue or Ctrl-C to cancel." before changing anything, unless called with `--skip-prompt`, which means "part of an install or automation: ask nothing"
  - run `sudo /usr/bin/apt update` before `apt install`
  - pin any third-party apt repo to its own packages in `/etc/apt/preferences.d/`, and have the `-remove` script delete the same pin file
- Any yes/no question that follows "Press Enter" defaults to No (`[y/N]`), so a reflexive second Enter changes nothing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dougburks/ohmydebn](https://github.com/dougburks/ohmydebn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
