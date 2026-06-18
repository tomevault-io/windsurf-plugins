---
trigger: always_on
description: This is an **Ansible-based Windows dotfile and workstation provisioning project**.
---

## Project Identity

This is an **Ansible-based Windows dotfile and workstation provisioning project**.
It manages a single Windows machine (or multiple) from a macOS/Linux control node via WinRM,
or locally via WSL2. It is a ground-up rewrite of the bare-git `sync-windows` pattern — see
`/Users/div/Projects/sync-windows` for the original.

Target machine profile: Windows 10/11 personal workstation. Primary user: Divit Mittal.

---

## Architecture

### Control node → target flow

```
macOS (ansible-playbook) ──WinRM──▶ Windows machine
        OR
WSL2 on Windows (local connection) ──▶ Windows machine
```

Connection is configured per-host in `inventory/hosts.yml`. Default is `ansible_connection: local`
(WSL2). For remote, switch to `winrm` and run `scripts/bootstrap-winrm.ps1` first on the target.

### Role dependency chain

```
bootstrap  ←── scoop  ←── msys2
               (no dep)    winget
windows_settings
dotfiles  ←── windows_settings
```

`meta/main.yml` in each role declares these. Ansible resolves them automatically — do not manually
re-include dependent roles in playbooks.

### Variable precedence (high → low)

```
host_vars/windows_workstation.yml
  group_vars/windows/packages.yml
  group_vars/windows/main.yml
    group_vars/all/vault.yml        ← encrypted, contains git_email / Windows creds
    group_vars/all/main.yml
      role defaults/main.yml
```

Never read `vault_*` variables directly in tasks. They are aliased to clean names in
`group_vars/windows/main.yml` (e.g. `git_email: "{{ vault_git_email }}"`).

---

## Key Files

| File | Purpose |
|------|---------|
| `ansible.cfg` | WinRM transport, fact caching (1h TTL), yaml callback, vault path |
| `inventory/hosts.yml` | Host definitions — edit `ansible_host` for remote targets |
| `inventory/group_vars/windows/packages.yml` | **All packages** — Scoop, Winget, MSYS2 lists |
| `inventory/group_vars/windows/main.yml` | All Windows path variables (scoop_dir, appdata, etc.) |
| `inventory/group_vars/all/vault.yml` | Encrypted secrets — edit with `make edit-vault` |
| `inventory/host_vars/windows_workstation.yml` | Per-machine overrides (is_laptop, extra packages) |
| `Makefile` | Primary interface — `make help` lists all targets |
| `scripts/bootstrap-winrm.ps1` | Run on Windows as Admin before first remote Ansible run |
| `scripts/install-ansible.sh` | Set up macOS control node (venv, collections, vault) |

---

## Roles Reference

### `bootstrap`
- Sets PowerShell execution policy, NuGet/PSGallery trust
- Installs Scoop if absent (with `block/rescue` error handling)
- Optionally configures WinRM (only when `bootstrap_configure_winrm: true`)
- Creates XDG base directories on the Windows target

### `scoop`
- Adds buckets in order: main → extras → nerd-fonts → versions
- Uses `set_fact` to compute missing packages before looping — avoids shelling out for each
- `block/rescue` around the install loop: collects failures, warns, does not abort

### `winget`
- Handles winget's non-standard exit code `-1978335189` (already installed) in `failed_when`
- Skips packages already present via string search on `winget list` output

### `msys2`
- Runs all pacman commands via `bash.exe -l -c` to load the MSYS2 environment
- Depends on `scoop` (msys2 is installed as a Scoop app)

### `windows_settings`
- **`registry.yml`**: win_regedit for Developer Mode, telemetry, context menu, taskbar, power plan
- **`environment.yml`**: win_environment for XDG vars, EDITOR, VISUAL, SHELL, GIT_CONFIG_GLOBAL
- **`path.yml`**: win_path for Scoop shims, .local/bin, MSYS2 bin — asserts PATH after update
- **`explorer.yml`**: Explorer advanced options loop (show extensions, hidden files, launch target)
- Handlers: `restart explorer`, `refresh env` — triggered via `notify`/`listen`

### `dotfiles`
- **Static files** (`roles/dotfiles/files/`): deployed with `win_copy` — wezterm Lua, tridactylrc, whkdrc, fastfetch config, git attributes/ignore
- **Jinja2 templates** (`roles/dotfiles/templates/`): deployed with `win_template`
  - `git/config.j2` — injects `user_name`, `git_email` (from vault)
  - `Profile.ps1.j2` — injects editor, shell, paths; feature flags gate MSYS2/starship sections
  - `starship.toml.j2` — `is_laptop` controls battery module; `feature_msys2` controls shell indicator
  - `firefox/user.js.j2` — privacy hardening prefs, no per-user vars currently
- Firefox profile path is discovered at runtime via `win_find` (not hardcoded)

---

## Feature Flags

All flags live in `group_vars/all/main.yml`. Set to `false` to skip entire subsystems:

```yaml
feature_scoop:            true/false
feature_winget:           true/false
feature_msys2:            true/false
feature_wezterm:          true/false
feature_starship:         true/false
feature_firefox:          true/false
feature_tridactyl:        true/false
feature_whkd:             true/false
feature_kanata:           false     # legacy, off by default
feature_windows_settings: true/false
feature_git_config:       true/false
```

Per-host variations (e.g. `is_laptop: true`) go in `host_vars/<hostname>.yml`.

---

## Collections Required

```
ansible.windows    >=2.0.0   — win_regedit, win_environment, win_path, win_service, etc.
community.windows  >=2.0.0   — win_scoop (if used directly)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DivitMittal/playbooks-4-windows](https://github.com/DivitMittal/playbooks-4-windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
