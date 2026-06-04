---
trigger: always_on
description: - **Repository Type**: Personal dotfiles managed with GNU Stow
---

# Agent Guidelines for Dotfiles Repository

## Critical Context
- **Repository Type**: Personal dotfiles managed with GNU Stow
- **No Build System**: Configuration files only, no tests/linting
- **Platforms**: macOS (primary), Ubuntu Server (secondary)
- **Shells**: Fish (primary) → Zsh (secondary) → Bash (fallback)
- **Editors**: Neovim (triple setup: Kickstart + LazyVim + AstroNvim) + Helix (secondary)
- **Profiles**: `shared/` (all), `personal/` (macOS), `work/` (macOS), `remote-sandbox/` (remote Linux base), `rca/` (RCA overlay), `crafting/` (crafting.dev overlay), `ubuntu-server/` (Ubuntu)

### Stow Commands
```bash
stow .                                 # Install all
stow shared personal                   # Install specific profiles (macOS)
stow shared ubuntu-server              # Install specific profiles (Ubuntu)
stow shared remote-sandbox rca         # RCA machine
stow shared remote-sandbox crafting    # Crafting.dev sandbox
stow -R .                              # Restow (re-link)
stow -D .                              # Uninstall
```

## Troubleshooting

When helping debug issues with tools in this repository:
1. **Check first**: Read `TROUBLESHOOTING.md` for known issues and fixes
2. **Document fixes**: After resolving a new issue, add it to `TROUBLESHOOTING.md` with symptom/cause/fix

## Shell Configuration Architecture

### CRITICAL RULE: Zero Duplication
**Define once, source everywhere.** Changes must work across ALL shells: Bash, Zsh, Fish.

### Shell Configuration Map
| File | Purpose | Sources |
|------|---------|---------|
| `.profile` | POSIX environment (PATH, exports) | - |
| `.profile.d/*.sh` | Profile-specific `.profile` extensions | - |
| `.config/shell/exports.sh` | Shared environment variables | - |
| `.config/shell/aliases.sh` | Shared aliases (POSIX) | - |
| `.config/shell/functions.sh` | Shared functions (POSIX) | - |
| `.bash_profile` | Bash login shell | `.profile`, `.bashrc` |
| `.bashrc` | Bash interactive | `shell/{exports,aliases,functions}.sh` |
| `.bashrc.d/*.sh` | Profile-specific `.bashrc` extensions | - |
| `.zshenv` | Zsh environment | `.profile` |
| `.zshrc` | Zsh interactive | `shell/{exports,aliases,functions}.sh` |
| `.zshrc.d/*.sh` | Profile-specific `.zshrc` extensions | - |
| `.zprofile` | Zsh login shell | - |
| `.zprofile.d/*.sh` | Profile-specific `.zprofile` extensions | - |
| `fish/config.fish` | Fish (self-contained) | Fish-specific equivalents |

### Decision Tree for Configuration Changes
```
New configuration needed?
├─ Environment variable → `.config/shell/exports.sh` + Fish equivalent
├─ Alias/Function → Is it profile-specific?
│  ├─ YES → `{profile}/.config/shell/aliases.{profile}.sh` + Fish equivalent
│  └─ NO → `.config/shell/aliases.sh` + Fish equivalent
├─ Interactive feature → Shell-specific rc file only
└─ Profile-specific shell init logic (not an alias/export/function)?
   └─ `{profile}/.bashrc.d/{profile}.sh` / `.zshrc.d/` / `.profile.d/` / `.zprofile.d/`
```

### Multi-Shell Requirements (NON-NEGOTIABLE)
**Task incomplete until implemented in ALL shells: Bash, Zsh, Fish**

1. Add to POSIX shells: `.config/shell/*.sh`
2. Add Fish equivalent: `.config/fish/config.fish` or `functions/*.fish`
3. Test in all three shells
4. Document shell-specific workarounds if needed

### PATH Priority vs Homebrew (nix-darwin)
Nix-darwin generates system shell configs (`/etc/fish/config.fish`, `/etc/zshrc`, `/etc/bashrc`) that run `brew shellenv`, which prepends `/opt/homebrew/bin` to PATH. This runs **after** the initial PATH setup in `exports.sh`/`env.fish` but **before** user rc files (`.bashrc`/`.zshrc`/`config.fish`).

To ensure custom paths (go, cargo, etc.) take priority over Homebrew:
- **Fish**: `fish_add_path` must use `--move` flag (without it, existing paths aren't repositioned)
- **Bash/Zsh**: `.bashrc`/`.zshrc` re-source `exports.sh` to re-prepend custom paths after `brew shellenv`

When adding new PATH entries that should beat Homebrew, ensure they follow this pattern in all three shells.

### CI Performance Tracking
**When modifying shell startup:** Update `.github/workflows/shell-performance.yml`

**Startup dependencies** (auto-run on shell init): oh-my-posh, zoxide, fzf, basher, zinit
- Add new tools to CI "Install shell startup tools" step
- Remove from CI when lazy-loading or removing tools

## Code Style (Quick Reference)
| Language | Style |
|----------|-------|
| **Shell** | `#!/bin/sh`, `${var}` format, `[[ ]]` for bash tests |
| **Lua** | 2-space indent, 120 char width, return tables, `-- stylua: ignore` to skip format |
| **TOML** | 2-space indent, lowercase-with-hyphens keys |

## File Organization
```
.config/
├── nvim/lua/custom/plugins/   # Kickstart Neovim plugins (default config)
├── lazyvim/lua/plugins/       # LazyVim plugins
├── astronvim/lua/plugins/     # AstroNvim plugins
├── fish/functions/            # Fish functions
├── shell/                     # Shared POSIX configs (exports, aliases, functions)
└── opencode/agents/           # ⚠️ MUST be 'agents/' plural (not 'agent/' - conflicts with Copilot)

{profile}/.config/
├── shell/aliases.{profile}.sh # Profile-specific POSIX configs
└── fish/config.{profile}.fish # Profile-specific Fish configs

{profile}/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josephschmitt/dotfiles](https://github.com/josephschmitt/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
