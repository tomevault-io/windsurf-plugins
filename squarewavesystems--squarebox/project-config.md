---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

squarebox is a containerized development environment (Docker or Podman) combining modern CLI/TUI tools with Claude Code. It uses a persistent container model — the container suspends on exit and resumes on restart, preserving state. Workspace code lives on the host at `~/squarebox/workspace` via volume mount.

## Build & Run

```bash
# Build the image (docker or podman — both work)
docker build -t squarebox .

# Create and run a new container (SSH agent forwarding, capability-restricted)
docker run -it --name squarebox \
  --cap-drop=ALL --cap-add=CHOWN --cap-add=DAC_OVERRIDE \
  --cap-add=FOWNER --cap-add=SETUID --cap-add=SETGID --cap-add=KILL \
  -e SSH_AUTH_SOCK=/tmp/ssh-agent.sock \
  -v "$SSH_AUTH_SOCK:/tmp/ssh-agent.sock" \
  -v ~/.ssh/config:/home/dev/.ssh/config:ro \
  -v ~/.ssh/known_hosts:/home/dev/.ssh/known_hosts:ro \
  -v ~/squarebox/workspace:/workspace \
  -v ~/.config/git:/home/dev/.config/git \
  -v ~/squarebox/.config/starship.toml:/home/dev/.config/starship.toml \
  -v ~/squarebox/.config/lazygit:/home/dev/.config/lazygit \
  -v /etc/localtime:/etc/localtime:ro \
  squarebox

# Resume an existing container
docker start -ai squarebox
```

Replace `docker` with `podman` above if using Podman. The `install.sh` script auto-detects the runtime; override with `SQUAREBOX_RUNTIME=docker|podman`.

The `install.sh` script automates initial setup (clone, build, create container, add `sqrbx` shell alias). A `.devcontainer/devcontainer.json` is also provided for VS Code Dev Containers / Codespaces.

**Windows PowerShell**: Only PowerShell 7+ (`pwsh`) is supported. Windows PowerShell 5.1 is not supported. `install.ps1` is the recommended Windows entry point — it handles clone, build, container creation, and PowerShell profile setup natively without requiring Git Bash. Running `install.sh` directly from Git Bash still works but only sets up bash aliases; it prints instructions to run `install.ps1` for PowerShell.

## First-Run Setup

`setup.sh` runs automatically on first container launch and prompts for:

1. **Git identity** — name and email (skipped if already configured)
2. **GitHub CLI auth** — persisted to `/workspace/.squarebox/gh` across rebuilds
3. **AI coding assistant** — Claude Code, GitHub Copilot CLI, Google Gemini CLI, OpenAI Codex CLI, OpenCode (any combination)
4. **Text editors** — micro, edit (Microsoft), fresh, nvim (nano is always available)
5. **TUI tools** — lazygit, gh-dash, yazi (any combination)
6. **Terminal multiplexers** — tmux, zellij
7. **SDKs** — Node.js (via nvm), Python (via uv), Go, .NET, Rust (via rustup)
8. **Shell** — bash (default) or zsh + Oh My Zsh + autosuggestions + syntax highlighting (experimental)

Selections are saved to `/workspace/.squarebox/` and reused on subsequent rebuilds.

### Re-running Setup

`sqrbx-setup` re-runs the setup wizard inside a running container to add or change tool selections:

```bash
sqrbx-setup                  # Re-run all sections
sqrbx-setup ai editors       # Re-run specific sections only
sqrbx-setup --list            # Show current tool selections
sqrbx-setup --help            # Show usage
```

Valid sections: `git`, `github`, `ai`, `editors`, `tuis`, `multiplexers`, `sdks`, `shell`.

Run `source ~/.bashrc` after setup to pick up new aliases and PATH changes in the current shell.

## Updating Tool Versions

```bash
# Update pinned Dockerfile-tier versions, checksums, and Dockerfile ARGs
scripts/update-versions.sh

# Inside a running container, update tool binaries in-place from GitHub releases
sqrbx-update
```

`scripts/update-versions.sh` only touches the Dockerfile tier (delta, yq, xh, glow, gum, starship, just, difftastic). It fetches latest GitHub releases, downloads artifacts for both architectures, computes SHA256 checksums, and updates `checksums.txt` and the Dockerfile ARGs.

Optional tools installed by `setup.sh` (opencode, editors, TUIs, zellij, Go, nvm) are not pinned. They install the latest upstream release at setup time, so there is no checksum file or version variable to update in the repo.

## CI

GitHub Actions workflow (`.github/workflows/build.yml`) validates the Dockerfile builds on every push and PR using buildx with GitHub Actions cache. An E2E test suite (`scripts/e2e-test.sh`, `.github/workflows/e2e.yml`) runs the container and validates tool installations.

## Dockerfile Architecture

The Dockerfile (Ubuntu 24.04 base) is organized into sequential stages:

1. **Base packages** — git, curl, ripgrep, bat, fzf, zoxide, fd, etc. via apt
2. **External APT repos** — GitHub CLI, Eza via apt (requires gnupg, stays combined)
3. **Per-tool binary installs** — one `RUN` layer per tool via `sb_install` from the shared library
4. **User setup** — creates non-root `dev` user with workspace directory
5. **Config files** — git config with delta as default pager (lazygit config is set up by setup.sh when lazygit is installed)
6. **Setup script & configs** — copies `setup.sh`, `sqrbx-update`, starship.toml
7. **Shell config** — bashrc with starship prompt, zoxide, aliases


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SquareWaveSystems/squarebox](https://github.com/SquareWaveSystems/squarebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
