---
trigger: always_on
description: **ALWAYS use GitHub API for external references:**
---

# Copilot Instructions for finpilot bootc Image Template

## CRITICAL: GitHub API Usage

**ALWAYS use GitHub API for external references:**
- When researching other repositories (e.g., projectbluefin/distroless, ublue-os/bluefin)
- When checking Containerfiles, build scripts, or configuration files
- Use the `github-mcp-server-get_file_contents` tool instead of curl/wget
- This ensures consistent, authenticated access and better error handling

## CRITICAL: Pre-Commit Checklist

**Execute before EVERY commit:**
1. **Conventional Commits** - ALL commits MUST follow conventional commit format (see below)
2. **Shellcheck** - `shellcheck *.sh` on all modified shell files
3. **YAML validation** - `python3 -c "import yaml; yaml.safe_load(open('file.yml'))"` on all modified YAML
4. **Justfile syntax** - `just --list` to verify
5. **Confirm with user** - Always confirm before committing and pushing

**Never commit files with syntax errors.**

### REQUIRED: Conventional Commit Format

**ALL commits MUST use conventional commits format**

```
<type>[optional scope]: <description>
```

## CRITICAL: Template Initialization

**When this repository is used as a template, you MUST:**

### 1. Rename ALL instances of `finpilot`

**Source of truth**: `Containerfile` line 9: `# Name: finpilot`

**Files to update:**
- `Containerfile` (line 9)
- `Justfile` (line 1)
- `README.md` (line 1)
- `artifacthub-repo.yml` (line 5)
- `custom/ujust/README.md` (~line 175)
- `.github/workflows/ghcr-pruner.yml` (line 22)

### 2. Create "What's Different" section in README

**CRITICAL**: Add this section near the top of README.md (after the title/intro, before detailed docs):

```markdown
## What Makes this Raptor Different?

Here are the changes from [Base Image Name]. This image is based on [Bluefin/Bazzite/Aurora/etc] and includes these customizations:

### Added Packages (Build-time)
- **System packages**: tmux, micro, mosh - [brief explanation of why]

### Added Applications (Runtime)
- **CLI Tools (Homebrew)**: neovim, helix - [brief explanation]
- **GUI Apps (Flatpak)**: Spotify, Thunderbird - [brief explanation]

### Removed/Disabled
- List anything removed from base image

### Configuration Changes
- Any systemd services enabled/disabled
- Desktop environment changes
- Other notable modifications

*Last updated: [date]*
```

**Maintenance requirement**: 
- **ALWAYS update this section when you modify packages or configuration**
- Keep descriptions brief and user-focused (explain "why", not just "what")
- Write for typical Linux users, not developers
- Update the "Last updated" date with each change

### 3. Verify copilot instructions are present

Confirm `.github/copilot-instructions.md` exists in the new repository. This file should be automatically copied when using this as a GitHub template.

### 4. Explain signing is optional

Signing is DISABLED by default. First builds succeed immediately. Enable later for production (see README).

**These 4 steps are REQUIRED for every new template instance.**

---

## Repository Structure

```
├── Containerfile          # Main build definition (multi-stage build with OCI imports)
├── Justfile              # Local build automation (image name, build commands)
├── build/                # Build-time scripts (10-build.sh, 20-chrome.sh, etc.)
│   ├── 10-build.sh      # Main build script (copy custom files, install packages)
│   ├── 20-*.sh.example  # Example third-party repos (rename to use)
│   ├── 30-*.sh.example  # Example desktop replacement (rename to use)
│   ├── copr-helpers.sh  # Helper functions for COPR repositories
│   └── README.md        # Build scripts documentation
├── custom/               # User customizations (NOT in container, installed at runtime/first boot)
│   ├── brew/            # Homebrew Brewfiles (CLI tools, dev tools)
│   │   ├── default.Brewfile      # General CLI tools
│   │   ├── development.Brewfile  # Dev environments
│   │   ├── fonts.Brewfile        # Font packages
│   │   └── README.md             # Homebrew documentation
│   ├── flatpaks/        # Flatpak preinstall (GUI apps, post-first-boot)
│   │   ├── default.preinstall    # Default GUI apps (INI format)
│   │   └── README.md             # Flatpak documentation
│   └── ujust/           # User commands (shortcuts to Brewfiles, system tasks)
│       ├── custom-apps.just      # App installation shortcuts
│       ├── custom-system.just    # System maintenance commands
│       └── README.md             # ujust documentation
├── iso/                  # Local testing only (no CI/CD)
│   ├── disk.toml        # VM/disk image config (QCOW2/RAW)
│   ├── iso.toml         # ISO installer config (bootc switch URL)
│   └── rclone/          # Upload configs (Cloudflare R2, AWS S3, etc.)
├── .github/              # GitHub configuration and CI/CD
│   ├── workflows/       # GitHub Actions workflows
│   │   ├── build.yml               # Builds :stable on main
│   │   ├── clean.yml               # Deletes images >90 days old
│   │   ├── renovate.yml            # Renovate bot updates (6h interval)
│   │   ├── validate-*.yml          # Pre-merge validation checks
│   │   └── ...
│   ├── copilot-instructions.md  # THIS FILE - Instructions for Copilot
│   ├── SETUP_CHECKLIST.md       # Quick setup checklist for users

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectbluefin/finpilot](https://github.com/projectbluefin/finpilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
