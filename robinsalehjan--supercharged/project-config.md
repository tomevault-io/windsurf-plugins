---
trigger: always_on
description: macOS environment setup automation — installs dev tools (Homebrew, ASDF), manages dotfiles, and backs up Claude Code configuration.
---

# AGENTS.md

macOS environment setup automation — installs dev tools (Homebrew, ASDF), manages dotfiles, and backs up Claude Code configuration.

This file is the canonical reference for AI agents and contributors. See [README.md](./README.md) for user-facing documentation and [SECURITY.md](./SECURITY.md) for security policy.

## Project Structure

See [README.md](./README.md) for detailed project structure. Key directories:
- `scripts/` - Shell scripts (mac.sh, update.sh, utils.sh, restore.sh, setup-profile.sh, help.sh, install-plugins.sh; backup-claude.sh/restore-claude.sh for Claude config)
- `dot_files/` - Dotfiles copied to `$HOME`
- `claude_config/` - Claude Code config backup
- `agent_config/` - Shared global agent instructions restored to both Claude and Codex
- `codex_config/` - Codex CLI/IDE config backup

## Code Conventions

- **Logging**: Use `log_with_level "INFO|WARN|ERROR|SUCCESS" "message"` from `utils.sh`
- **Error handling**: Include `trap cleanup EXIT` in scripts
- **Tests**: BATS (Bash Automated Testing System); test files in `tests/`; use `setup_test_env` + `teardown_test_env`
- **Testing workflow**: Add BATS tests in `tests/<script-name>/` for new script features; run `npm test` before committing script changes
- **Commits**: Conventional format preferred (not enforced). Scope optional.
  - Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci`, `revert`, `wip`, `update`, `add`, `remove`
  - Examples: `feat(backup): add new feature`, `fix(restore): handle edge case`, `chore(deps): bump versions`
- **Shell scripts**: Written for zsh; ShellCheck `--shell=bash` flags (SC1071, SC2296) are safe to ignore
- **Dotfiles**: Use env vars, no hardcoded paths

## npm Commands Reference

```bash
# Setup and Installation
npm run setup                 # Fresh install (interactive)
npm run restore:dotfiles         # Copy dotfiles + Claude config to $HOME

# Updates
# npm run update runs: backup:claude → restore:dotfiles → update.sh
npm run update                    # Update all components (brew, asdf, zsh, npm, pip)
npm run update:dry-run            # Preview outdated brew/npm packages (read-only)
npm run update:only -- <comp>     # Copy dotfiles + update one component (brew, asdf, zsh, npm, pip)

# Validation and Recovery
npm run validate              # Verify all tools installed correctly
npm run restore               # Restore from latest backup

# Claude Code Configuration
npm run backup:claude             # Backup Claude Code config to repo
npm run restore:claude            # Restore Claude Code config (only if repo is newer)
npm run restore:claude -- --force # Force restore Claude Code config (see Post-Restore Steps below)
npm run backup:all                # Backup Claude Code config + Codex config in one step
npm run backup:codex              # Backup Codex config to repo
npm run restore:codex             # Restore Codex config (only if repo is newer)
npm run restore:codex -- --force  # Force restore Codex config
npm run restore:all               # Restore Claude config + Codex config + dotfiles in one step
npm run install:plugins           # Install all marketplaces and plugins via claude CLI
npm run install:plugins -- --dry-run # Preview what would be installed
npm run install:skills            # Clone/update git-based skills into ~/.claude/skills
npm run install:skills -- --dry-run # Preview what would be installed/updated

# Versioning and Releases
npm run version:show              # Print version, commit SHA, tag, branch, host
npm run release -- patch          # Cut a release: bump, commit, tag vX.Y.Z, push
npm run release -- minor          # Minor bump
npm run release -- 1.2.3          # Explicit version
npm run release -- --dry-run patch # Preview without making changes

# Development
npm run lint                      # ShellCheck all scripts (including utils/)
npm run scan:secrets              # Scan repository paths for likely secrets
npm test                          # Run all BATS tests
npm run test:watch                # Re-run tests on change (requires nodemon)
bats tests/<suite>/*.bats        # Run a specific suite (claude, utils, mac, update, setup, restore, meta)
npm run help                      # Display all available commands
```

## ShellCheck Notes

**Safe warnings** (zsh scripts run through `--shell=bash`):
SC1071 (zsh unsupported), SC2296 (zsh `${(%):-%x}`), SC1091 (sourced file), SC2001 (sed vs expansion). These are disabled centrally in `.shellcheckrc` so the rule list lives next to the code rather than buried in `package.json`. Also safe but not disabled: SC2155 (declare+assign), SC2012 (ls vs find).

`npm run lint` runs with `--severity=warning` to keep `info`/`style` notes from cluttering output without changing what the rule set considers failing.

**Zsh-specific syntax** used in scripts:
`${(%):-%x}`, `${(%):-%n}`, `&!` (disown), `path=(...)`, `setopt`.

## Testing

### BATS Testing Infrastructure

This project uses [BATS (Bash Automated Testing System)](https://github.com/bats-core/bats-core) for testing shell scripts.

**Running tests:**
```bash
npm test                          # Run all BATS tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robinsalehjan/supercharged](https://github.com/robinsalehjan/supercharged) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
