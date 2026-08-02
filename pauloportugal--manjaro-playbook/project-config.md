---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible playbook for configuring Manjaro/Arch Linux GNOME 3 desktop systems. It's designed to run locally after a clean OS install and follows Manjaro's community recommendations for package management (pacman for official packages, Pamac for AUR packages).

## Required Variables

The playbook requires three user-specific variables passed via `--extra-vars`:
- `user_name`: The system username
- `user_git_name`: Git user name for configuration
- `user_email`: Git email for configuration

These are NOT defined in defaults and must always be provided when running the playbook.

## Development Setup

### Initial Setup
```bash
# Run the automated setup script (installs all dependencies)
./setup-dev-environment.sh
```

This installs: ansible, ansible-lint, git, xclip, python-pip, yamllint, python-pytokens, shellcheck, pre-commit, and Ansible Galaxy collections.

### Code Quality Checks

Pre-commit hooks are automatically installed by `setup-dev-environment.sh` and run on every commit.

Manual validation:
```bash
# Run all quality checks
yamllint . && ansible-lint

# Check playbook syntax
ansible-playbook --syntax-check playbook.yml

# Check shell scripts
shellcheck tests/*.sh aur/*.sh setup-dev-environment.sh
```

## Running the Playbook

### Full Installation
```bash
ansible-playbook playbook.yml -l localhost \
  --extra-vars="user_name=USERNAME user_git_name=GIT_USERNAME user_email=EMAIL" \
  --ask-become-pass
```

### Run Specific Role with Tags
```bash
ansible-playbook playbook.yml -l localhost \
  --extra-vars="user_name=USERNAME user_git_name=GIT_USERNAME user_email=EMAIL" \
  --ask-become-pass \
  --tags browsers
```

Available tags match role names: `base`, `users`, `printers`, `browsers`, `development`, `cloud`, `editors`, `media`, `multimedia`, `audio`, `comms`, `gnome`, `security`, `virtualization`

Note: The `users` role has a `never` tag and must be explicitly called.

### Debug Mode
```bash
# Verbose output
ansible-playbook -v playbook.yml ...

# Maximum verbosity
ansible-playbook -vvvv playbook.yml ...
```

## Testing

### Run All Tests
```bash
./tests/run-all-tests.sh
```

This runs:
1. Code quality checks (yamllint, ansible-lint, syntax check, shellcheck)
2. Unit tests (if on testbuild host)
3. Integration tests (if Vagrant VM is running)

### Verification Tests
```bash
# Run verification against localhost
ansible-playbook tests/verify.yml --connection=local -i localhost,

# Run specific verification tags
ansible-playbook tests/verify.yml --connection=local -i localhost, --tags verify-base
```

Available verification tags: `verify-base`, `verify-development`, `verify-security`, `verify-browsers`, `verify-editors`, `verify-config`

### Idempotency Testing
```bash
# Test specific target
./tests/test-idempotency.sh testbuild

# Test specific role
./tests/test-idempotency.sh testbuild base

# Test with skipped tags
./tests/test-idempotency.sh localhost --skip-tags aur
```

### Role-Specific Tests
```bash
# Test individual role functionality and idempotency
ansible-playbook tests/test-roles.yml --connection=local -i localhost, --tags test-base
```

## Architecture

### Role Structure

The playbook follows standard Ansible role structure:
```
roles/
  <role-name>/
    tasks/main.yml       # Primary task entry point
    tasks/*.yml          # Subtasks imported by main.yml
    defaults/main.yml    # Default variables
    files/               # Static files to copy
    templates/           # Jinja2 templates
    handlers/            # Handlers for this role
```

### Variable Precedence

Critical pattern for user-specific paths:
- Roles use `{{ user_name }}` variable for paths like `/home/{{ user_name }}/...`
- Test playbooks set: `user_name: "{{ ansible_user_id }}"` when running standalone
- Verification playbook (tests/verify.yml) sets `user_name` to `ansible_user_id` if not already defined
- This allows roles to work both in production (with user_name passed) and testing (using ansible_user_id)

### Package Management Strategy

The playbook uses multiple package managers:
1. **pacman**: For official Arch Linux packages (via `community.general.pacman` module)
2. **Pamac**: For automated AUR package installation (via `shell` commands)
3. **Custom AUR script**: `aur/install-aur.sh` for manual AUR installations

### Workspace Directory

The base role creates `~/Workspace` directory. This is a standard convention:
- Created by: `roles/base/tasks/main.yml`
- Path: `/home/{{ user_name }}/Workspace`
- Shell alias: `W='cd ~/Workspace'` (defined in base role defaults)

## CI/CD

### CI Workflow

GitHub Actions workflow (`.github/workflows/ci.yml`) runs:

1. **Lint Job**: yamllint, ansible-lint, syntax checks, trailing whitespace check
2. **Security Job**: ShellCheck on shell scripts
3. **Test Job**: Syntax verification and verification tests (with test environment setup)

The workflow matrix includes:
- "Syntax Verification": `ansible-playbook --syntax-check playbook.yml`
- "Verification Tests": `ansible-playbook tests/verify.yml --connection=local -i localhost,`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PauloPortugal/manjaro-playbook](https://github.com/PauloPortugal/manjaro-playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
