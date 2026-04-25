---
trigger: always_on
description: <!-- This file is canonical. CLAUDE.md is a symlink to AGENTS.md. -->
---

# Agent Instructions

<!-- This file is canonical. CLAUDE.md is a symlink to AGENTS.md. -->

This document provides guidance for AI agents working in this repository. Claude agents also receive contextual rules (`.claude/rules/`) and skills (`.claude/skills/`) auto-loaded when relevant. All agents should run `cargo xtask help` to discover build commands.

Codex-specific repo skills live in `.codex/skills/`. Prefer them when the task matches:
- `nteract-daemon-dev` for per-worktree daemon lifecycle, socket setup, and daemon-backed verification
- `nteract-python-bindings` for `maturin develop`, venv selection, and MCP server work
- `nteract-notebook-sync` for Automerge ownership, output manifests, and sync-path changes
- `nteract-testing` for choosing and running the right verification path

## Development Setup Requirements

### direnv (Required for Daemon Isolation)

**direnv is required** to automatically set `RUNTIMED_DEV=1` and `RUNTIMED_WORKSPACE_PATH` when you enter the repo directory. Without it, the dev daemon and system nightly daemon will conflict, and nteract-dev MCP will connect to the wrong daemon.

**Install and configure:**
```bash
# Install direnv
sudo apt install direnv  # Ubuntu/Debian
brew install direnv      # macOS

# Add hook to shell (bash)
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
source ~/.bashrc

# Allow the repo's .envrc
cd /path/to/nteract/desktop
direnv allow
```

The `.envrc` file sets:
- `PATH_add bin` — adds `bin/runt` wrapper to PATH (shadows system `runt`)
- `export RUNTIMED_DEV=1` — enables per-worktree daemon isolation
- `export RUNTIMED_WORKSPACE_PATH="$(pwd)"` — pins daemon to this worktree

**Verify it works:**
```bash
cd /path/to/nteract/desktop
echo $RUNTIMED_DEV              # Should print "1"
echo $RUNTIMED_WORKSPACE_PATH   # Should print the repo path
which runt                      # Should be repo/bin/runt (not /usr/local/bin/runt)
```

### MCP Server Configuration

**Three MCP servers** should be configured:

1. **nteract-dev** (development, per-worktree daemon)
   - Command: `cargo xtask run-mcp` (builds and runs the local `mcp-supervisor` binary)
   - Working directory: `/path/to/nteract/desktop`
   - Env vars: Inherits from shell (direnv provides RUNTIMED_DEV, RUNTIMED_WORKSPACE_PATH)
   - Socket: `~/.cache/runt-nightly/worktrees/{hash}/runtimed.sock`
   - Tools: 26 nteract tools (proxied from `runt mcp`) + 5 dev tools (`up`, `down`, `status`, `logs`, `vite_logs`)

2. **nteract-nightly** (system nightly daemon, for testing releases)
   - Command: `env -i HOME=$HOME /usr/local/bin/runt-nightly mcp`
   - Working directory: Can be anywhere (no repo dependency)
   - Env vars: **MUST use `env -i`** to strip RUNTIMED_DEV and RUNTIMED_WORKSPACE_PATH
   - Socket: `~/.cache/runt-nightly/runtimed.sock` (system socket, NOT worktrees/)
   - Tools: 26 nteract tools (no dev tools)

3. **nteract** (system stable daemon, for testing stable releases)
   - Command: `env -i HOME=$HOME /usr/local/bin/runt mcp`
   - Working directory: Can be anywhere
   - Env vars: **MUST use `env -i`** to strip dev env vars
   - Socket: `~/.cache/runt/runtimed.sock`
   - Tools: 26 nteract tools (no dev tools)

**Critical:** The `env -i` wrapper on nteract-nightly and nteract is required to prevent direnv's env vars from leaking into these MCP servers. Without it, they will incorrectly connect to the dev daemon instead of the system daemon.

**This system's nightly installation:**
- Binaries: `~/.local/share/runt-nightly/bin/{runtimed-nightly,runt-nightly,nteract-mcp-nightly}`
- Symlinks: `/usr/local/bin/{runtimed-nightly,runt-nightly,nteract-mcp-nightly}` → the install dir
- Daemon socket: `~/.cache/runt-nightly/runtimed.sock`
- Install command: `cargo xtask install-nightly` (refuses on macOS and when an app bundle is installed — see § `install-nightly` below)
- Version: Built from source, updated after each PR merge

**Verification steps:** See `.claude/rules/mcp-servers.md` § Verifying Daemon Isolation.

## Quick Recipes (Common Dev Tasks)

### If you have `up` / `down` / `status` tools — use them

If your MCP client provides `up`, `down`, `status`, `logs`, `vite_logs` (provided by `nteract-dev`), **prefer those over manual terminal commands**. `nteract-dev` manages the dev daemon lifecycle for you — no env vars, no extra terminals.

**Claude Code has nteract-dev locally** — the local dev environment connects Claude Code to the repo-local `nteract-dev` MCP entry via `cargo xtask run-mcp`. Codex app/CLI can use the same server when this repo's project-scoped `.codex/config.toml` is enabled in a trusted workspace. If your current environment does not expose these tools, use the manual `cargo xtask` commands below.

| Instead of… | Use… |
|-------------|------|
| `cargo xtask dev-daemon` (in a terminal) | `up` (idempotent: ensures daemon + child are healthy) |
| `maturin develop` (rebuild bindings) | `up rebuild=true` |
| `runt daemon status` (with env vars) | `status` |
| `runt daemon logs` | `logs` |
| `cargo xtask vite` | `up vite=true` |
| `cargo xtask notebook` (stop dev processes) | `down` (stops Vite; `daemon=true` also stops daemon) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nteract/desktop](https://github.com/nteract/desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
