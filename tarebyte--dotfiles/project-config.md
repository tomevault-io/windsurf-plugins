---
trigger: always_on
description: Personal dotfiles. **GNU stow**-managed — source tree is organized into packages (`common/`, `darwin/`, `codespaces/`) that mirror `$HOME`. Files live at their real names (no `dot_` prefix); stow creates symlinks from `$HOME` into the repo. Repo-management files (`README.md`, `AGENTS.md`, `LICENSE`, `script/`, `Makefile`, `templates/`) sit at the source root, outside every package, so stow never touches them. Per-OS gating is handled by which packages you stow.
---

# AGENTS.md

Personal dotfiles. **GNU stow**-managed — source tree is organized into packages (`common/`, `darwin/`, `codespaces/`) that mirror `$HOME`. Files live at their real names (no `dot_` prefix); stow creates symlinks from `$HOME` into the repo. Repo-management files (`README.md`, `AGENTS.md`, `LICENSE`, `script/`, `Makefile`, `templates/`) sit at the source root, outside every package, so stow never touches them. Per-OS gating is handled by which packages you stow.

## Commands

`make test` is the repo's test target — runs `shellcheck -x` on every script plus the functional suite in `script/test`. Run it after any script edit. `script/doctor` is a separate health check for the installed environment on the current host, not for the repo itself.

| Command | Purpose |
|---------|---------|
| `script/setup` | First-time entry point. Installs `stow` via Homebrew/apt if missing, then runs `make install`. Auto-run by Codespaces. |
| `script/doctor` | Health check — core tools, shell, git, mise, Neovim, tmux/TPM. Also checks that `~/.config/git/config` matches the template (warns on drift). |
| `make test` | Run `shellcheck -x` on every script under `script/` and then execute `script/test` (functional tests for `stow-package` and `setup-git-config`). |
| `make install` | Generate `~/.config/git/config` from the template (prompting for identity on first run), stow `common`. On macOS also stow `darwin`, install brew bundle, update fisher, chsh to fish. |
| `make regen-git-config` | Re-render `~/.config/git/config` from `templates/git-config.tmpl` + `~/.config/dotfiles/identity.env`. Run this after editing the template. |
| `make brew` | Re-run `brew bundle --global` after editing `darwin/.Brewfile`. |
| `make fisher` | Bootstrap fisher if needed and run `fisher update` after editing `common/.config/fish/fish_plugins`. |
| `make mise` | `mise trust && mise install` after editing `common/.config/mise/config.toml`. |
| `make clean` | `stow -D` every package — cleanly unlinks everything from `$HOME`. Does NOT delete `~/.config/git/config` or `~/.config/dotfiles/identity.env`. |
| `:Lazy sync` | Update/install Neovim plugins via lazy.nvim; commit `lazy-lock.json`. |

`make install` is also the recovery incantation — re-running it after a broken state is safe because `script/stow-package` handles pre-existing files by moving them aside, and every other step is idempotent.

Stow symlinks files from the package into `$HOME`, so editing `~/.config/fish/config.fish` edits `common/.config/fish/config.fish` transparently. `git diff` in the repo shows your change immediately — no `apply`, no `re-add`, no drift. When you add a *new* tracked file inside a package, run `make install` so stow creates its symlink in `$HOME`.

If you ever need to invoke `stow` manually (e.g. for dry-run debugging), always include `--no-folding` — see the paragraph below for why. Dry-run example: `stow -n -v --no-folding -t $HOME common`.

**Why `--no-folding`:** every `stow` invocation in this repo passes `--no-folding` (see the `STOW` variable in the Makefile). With folding enabled, stow creates a single directory-level symlink when a package subtree is new in `$HOME` — e.g. `~/.config/fish` would become a symlink pointing at `common/.config/fish`. That's minimal-symlink-count, but it means any runtime write a tool does inside `~/.config/fish/` (fish writing `fish_variables`, fisher writing `conf.d/` and `completions/`) silently propagates *through the symlink into the repo source*, polluting the stow package with untracked files. `--no-folding` makes every directory in `$HOME` a real directory containing individual per-file symlinks, so:

- Edits to *tracked* files (`config.fish`, `lazy-lock.json`, etc.) still go through their per-file symlinks into the repo source, exactly as before.
- *New* files written at runtime (`fish_variables`, `conf.d/foo.fish`, `completions/bar.fish`, whatever) land in real `$HOME` paths and never enter the repo. They're the same as any other per-machine file.
- Adding a new tracked file is deliberate: create it in the package, run `stow -R` (or `make install`), and the per-file symlink appears in `$HOME`.

Defense-in-depth: the root `.gitignore` also lists a handful of known runtime-state paths inside packages, so if `--no-folding` is ever accidentally dropped, those files still won't get committed.

**The one exception is git config**, which is generated at install time rather than stow-symlinked. See "Git identity layout" below.

### Scripts

The `Makefile` is a thin dispatcher; real logic lives in `script/` so each piece is independently testable and shellcheck-clean:

| Script | Purpose |
|---|---|
| `script/setup` | Bootstrap — installs `stow` if missing, then runs `make install`. `#!/bin/sh` (POSIX). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarebyte/dotfiles](https://github.com/tarebyte/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
