---
trigger: always_on
description: Manages Fish shell, Neovim, Git, mise, Starship, Atuin, and related tools. Uses 1Password for secrets.
---

# AGENTS.md — Chezmoi Dotfiles Repo

Chezmoi dotfiles repository managing development environment across macOS and Linux.
Manages Fish shell, Neovim, Git, mise, Starship, Atuin, and related tools. Uses 1Password for secrets.

---

## Read This First

- **Serena memories (`.serena/`) are the primary source of detailed conventions** — read all
  project memories before making significant changes
- If this file and a Serena memory contradict each other, the Serena memory wins on detail
- Use English by default; only switch languages when explicitly asked

---

## Repository Layout

```
chezmoi/
├── home/                    # chezmoi source root — maps directly to $HOME
│   ├── .chezmoiscripts/     # setup/apply scripts (cross-platform + darwin/ linux/)
│   ├── .chezmoidata/        # package declarations and template data
│   ├── dot_config/          # → ~/.config/
│   ├── dot_local/           # → ~/.local/
│   ├── private_dot_ssh/     # → ~/.ssh/ (mode 600)
│   └── .chezmoi.toml.tmpl  # chezmoi config template
├── .github/workflows/       # CI: lint, shellcheck, chezmoi-verify, changelog
├── site/                    # Astro/Starlight docs site
├── CHANGELOG.md             # managed by git-cliff via GitHub Actions
└── .serena/                 # Serena memories — primary source of conventions
```

**Key path mappings:** `dot_` → `.`, `private_` → mode 600, `empty_` → empty file,
`.tmpl` → Go template rendered at apply time, `encrypted_` → age-encrypted file.

---

## Build / Lint / Test Commands

This repo has **no unit-test suite**. Validation is linting and chezmoi render checks.

```bash
# Shellcheck plain (non-templated) scripts
shellcheck \
  .install-op-and-age.sh \
  home/.chezmoiscripts/run_before_00-write-age-identity.sh \
  home/.chezmoiscripts/run_once_after_25-setup-op-gh-plugin.sh \
  home/dot_local/bin/executable_oscar-update

# Verify CI bootstrap helper is safe
CI=1 bash .install-op-and-age.sh

# YAML lint
yamllint .github/workflows/
yamllint home/.chezmoidata/packages.yaml

# TOML syntax
python3 -c "import tomllib; tomllib.load(open('cliff.toml', 'rb')); print('OK')"
python3 -c "import tomllib; tomllib.load(open('home/dot_config/starship.toml', 'rb')); print('OK')"
python3 -c "import tomllib; tomllib.load(open('home/dot_config/atuin/config.toml', 'rb')); print('OK')"

# Chezmoi render sanity check (may fail locally if 1Password is unavailable)
chezmoi --source=home dump --format=json --exclude=encrypted > /dev/null

# Render a single template for inspection
chezmoi --source=home execute-template -f home/dot_gitconfig.tmpl
chezmoi --source=home execute-template -f home/.chezmoiscripts/run_onchange_after_20-install-cargo-packages.sh.tmpl
```

**Templated scripts (`.sh.tmpl`):** CI is authoritative. Local renders may fail without 1Password.
CI matrix: `linux-ubuntu-headless`, `linux-arch-personal`, `linux-fedora-headless`, `darwin-personal`.

**Apply dotfiles:**
```fish
chezmoi apply   # czap   chezmoi diff    # czdf
chezmoi edit    # czed   chezmoi add     # czad   chezmoi cd  # czcd
```

---

## Code Style

### Shell scripts
- **Shebang:** always `#!/bin/bash` — never `#!/bin/sh`
- **Strict mode:** always `set -eufo pipefail` on the line after the shebang
- Fail loudly; never silently swallow errors
- Guard optional tool calls: `command -v <tool> || return/exit`
- Use numeric prefixes to control order: `before_10`, `after_20`, etc.

### Chezmoi templates (`.tmpl` files)
- Machine variables: `.chezmoi.os`, `.osid`, `.personal`, `.headless`, `.ephemeral`, `.hostname`
- **Secrets in templates:** `{{ onepasswordRead "op://vault/item/field" }}`
- **Secrets in scripts:** `op read "op://vault/item/field"` (allowed at apply time)
- **Never in Fish config:** `op read` blocks shell startup — use `onepasswordRead` in `.tmpl` files
- **Ephemeral gating:** wrap CI-unsafe scripts with `{{ if not .ephemeral -}} ... {{ end -}}`;
  keep the shebang and `set -eufo pipefail` inside the guard

### YAML
- Lint config: `.yamllint.yaml`; max line length 120 (warning); `document-start` disabled
- Truthy values: `true` / `false` only — never `yes` / `no`; 2-space indentation

### TOML / Editor
- TOML: 2-space indentation; validate with `python3 -c "import tomllib; ..."`
- `.editorconfig`: UTF-8, LF, final newline, trim whitespace; default indent **tabs** size 2;
  YAML/TOML/JSON use **spaces** size 2

---

## Naming Conventions

| File prefix/suffix | Meaning |
|---|---|
| `dot_` | Maps to `.` in target (e.g. `dot_gitconfig` → `~/.gitconfig`) |
| `private_` | Installed with mode 600 |
| `empty_` | Creates an empty file |
| `.tmpl` | Go template, rendered by chezmoi at apply time |
| `encrypted_` | Age-encrypted file |
| `run_once_` | Script runs once per machine (hash-tracked) |
| `run_onchange_` | Script runs when its content changes |
| `run_before_` / `run_after_` | Script ordering (before/after file writes) |
| Numeric infix (`before_10`, `after_20`) | Ordering within a phase |

---

## Fish Shell Config Layout

Files under `home/dot_config/fish/`:
- `conf.d/` — modular config loaded alphabetically at shell start
- `aliases.fish` — tool replacements; `abbreviations.fish` — workflow shortcuts (preferred over aliases)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SolidRhino/dotfiles](https://github.com/SolidRhino/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
