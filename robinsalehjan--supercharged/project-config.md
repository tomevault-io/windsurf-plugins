---
trigger: always_on
description: Detailed reference for AI agents. For project overview, structure, commands, and conventions see [CLAUDE.md](./CLAUDE.md).
---

# AGENTS.md

Detailed reference for AI agents. For project overview, structure, commands, and conventions see [CLAUDE.md](./CLAUDE.md).

## npm Commands Reference

```bash
# Setup and Installation
npm run setup                 # Fresh install (interactive)
npm run setup:profile         # Copy dotfiles + Claude config to $HOME

# Updates
# npm run update runs: backup:claude → setup:profile → update.sh
# update:brew and update:asdf also copy dotfiles first (via setup:profile)
npm run update                # Update all components (brew, asdf, zsh, npm, pip)
npm run update:dry-run        # Preview outdated brew/npm packages (read-only)
npm run update:brew           # Copy dotfiles + update Homebrew (formulae + casks)
npm run update:asdf           # Copy dotfiles + update ASDF plugins and versions
npm run update:zsh            # Update only ZSH plugins
npm run update:npm            # Update only npm global packages
npm run update:pip            # Update only pip data science packages

# Validation and Recovery
npm run validate              # Verify all tools installed correctly
npm run restore               # Restore from latest backup

# Claude Code Configuration
npm run backup:claude         # Backup Claude Code config to repo
npm run restore:claude        # Restore Claude Code config (only if repo is newer)
npm run restore:claude:force  # Force restore Claude Code config

# Development
npm run lint                  # ShellCheck all scripts (ignore zsh warnings)
npm test                      # Run all BATS tests
npm run test:claude           # Run Claude backup/restore tests only
npm run test:utils            # Run utility function tests only
npm run test:mac              # Run mac.sh smoke tests only
npm run test:update           # Run update.sh smoke tests only
npm run test:setup            # Run setup-profile.sh smoke tests only
npm run help                  # Display all available commands
```

## ShellCheck Notes

**Safe warnings** (zsh scripts run through `--shell=bash`):
SC1071 (zsh unsupported), SC2296 (zsh `${(%):-%x}`), SC1091 (sourced file), SC2155 (declare+assign), SC2001 (sed vs expansion), SC2012 (ls vs find).

**Zsh-specific syntax** used in scripts:
`${(%):-%x}`, `${(%):-%n}`, `&!` (disown), `path=(...)`, `setopt`.

## Testing

### BATS Testing Infrastructure

This project uses [BATS (Bash Automated Testing System)](https://github.com/bats-core/bats-core) for testing shell scripts.

**Running tests:**
```bash
npm test                          # Run all BATS tests
npm run test:claude               # Claude backup/restore tests
npm run test:utils                # Utility function tests
npm run test:mac                  # mac.sh smoke tests
npm run test:update               # update.sh smoke tests
npm run test:setup                # setup-profile.sh smoke tests
npm test -- --filter "pattern"    # Run specific tests
```

**Test structure:**
- `tests/claude/backup.bats` - Tests for Claude Code backup sanitization
- `tests/claude/restore.bats` - Tests for Claude Code restore merging
- `tests/utils/portability.bats` - Tests for portable path handling
- `tests/mac/install.bats` - Smoke tests for mac.sh (validate_system, build_brewfile, install_homebrew, parse_tool_versions)
- `tests/setup/profile.bats` - Smoke tests for setup-profile.sh (dotfile copying, restoration points, version_gte)
- `tests/update/update.bats` - Smoke tests for update.sh (argument parsing, help, dry-run, unknown flags)
- `tests/helpers/setup.bash` - Test environment setup and teardown utilities
- `tests/helpers/assertions.bash` - jq-based JSON assertion utilities
- `tests/helpers/mocks.bash` - Command mocking utilities
- `tests/fixtures/` - Test data (JSON configs with `$HOME` placeholders)

**Key test patterns:**
- Use `setup()` for test initialization (calls `setup_test_env`)
- Use `teardown()` for cleanup (calls `teardown_test_env`)
- Load helpers: `load '../helpers/setup'`, `load '../helpers/assertions'`, `load '../helpers/mocks'`
- Test environment creates isolated temp directories with mocked `$HOME`
- Fixtures use `$HOME` placeholders, not hardcoded paths
- JSON assertions: `assert_json_field`, `assert_json_equals`
- Domain assertions: `assert_plugin_exists`, `assert_plugin_not_exists`, `assert_marketplace_exists`, `assert_marketplace_not_exists`
- Zsh-only functions tested via `run zsh -c "source script; function_call"` subprocess pattern
- PATH-based mocking with `MOCK_BIN_DIR` for zsh subprocesses (bash `export -f` doesn't propagate to zsh)

**Pre-commit integration:**
Tests run automatically via `.husky/pre-commit` hook after security checks (if `tests/` directory exists and `bats` is installed).

**CI integration:**
Tests run on push to main and pull requests via `.github/workflows/test.yml`.

### Manual Testing Workflows

**Pre-commit**:
1. `shellcheck --shell=bash scripts/*.sh`
2. Test script functions in isolation
3. Verify logging matches existing patterns

**Manual workflow**:
1. `npm run setup:profile` to copy dotfiles
2. `source ~/.zshrc` — verify no errors
3. `npm run validate` — check installations
4. If issues: `npm run restore`

**Validation checks** (from `utils.sh`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robinsalehjan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
