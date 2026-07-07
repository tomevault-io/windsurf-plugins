---
trigger: always_on
description: This is an Ansible-based infrastructure automation repository that manages Docker servers using the **Ansible Pull** approach. Servers automatically pull their configuration from this repository and apply changes when Git commits are detected.
---

# Copilot Instructions for Docker Infrastructure Repository

## Project Overview

This is an Ansible-based infrastructure automation repository that manages Docker servers using the **Ansible Pull** approach. Servers automatically pull their configuration from this repository and apply changes when Git commits are detected.

## Technology Stack

- **Ansible**: 2.20+ (Infrastructure as Code)
- **Python**: 3.8+ (Ansible runtime)
- **Docker**: Container platform managed by Ansible
- **Task**: Task runner / build tool (taskfile.dev)
- **YAML**: Configuration and playbook syntax
- **Bash**: Helper scripts
- **Bats**: Testing framework
- **GitHub Actions**: CI/CD pipeline

## Key Commands

### Important: Always Use Task Runner
When suggesting commands to the user, **always prefer `task` commands** from `Taskfile.yml` over raw commands. Only fall back to raw commands if no equivalent task exists. Run `task --list` to discover available tasks.

### Important: Always Use mise for Tool Management
When installing or managing tool versions (Python, Ansible, ansible-lint, yamllint, Bats, Task, etc.), **always use `mise`** instead of `pip install`, `apt install`, or other package managers. Tool versions are defined in `.mise.toml` and must stay in sync.
```bash
# Install all tools at versions defined in .mise.toml
mise install

# Check installed tool versions
mise ls

# Update a tool version (edits .mise.toml)
mise use pipx:ansible-lint@26.5.0
```

### Task Runner (Recommended)
```bash
# Show all available tasks
task --list

# Install all dependencies
task install

# Run all tests
task test

# Run linting and syntax checks
task ci:quick

# Run full CI pipeline locally
task ci:local

# Check what would change without applying
task ansible:check

# Show detailed help
task help
```

### Testing
```bash
# Run all Bats tests (with Task)
task test

# Run specific test suite
task test:lint
task test:syntax
task test:docker

# Run all Bats tests (without Task)
./tests/bash/run-tests.sh

# Run specific test
./tests/bash/run-tests.sh --test lint-test.bats

# Run in CI mode (for automated pipelines)
./tests/bash/run-tests.sh --ci
```

### Running Ansible
```bash
# Dry-run with Task
task ansible:check

# Apply configuration with Task
task ansible:pull

# Manual ansible-pull execution
sudo ansible-pull \
    --url https://github.com/DevSecNinja/docker.git \
    --checkout main \
    --directory /var/lib/ansible/local \
    --inventory ansible/inventory/hosts.yml \
    --extra-vars "target_host=$(hostname)" \
    --only-if-changed \
    ansible/playbooks/main.yml
```

### Local Development
```bash
# Install all dependencies (with Task)
task install

# Install required Ansible collections
ansible-galaxy collection install -r ansible/requirements.yml

# Check playbook syntax locally
ansible-playbook ansible/playbooks/main.yml --syntax-check
```

## Repository Structure

```
ansible/
├── requirements.yml         # External roles and collections
├── playbooks/
│   └── main.yml            # Main playbook for ansible-pull
├── inventory/
│   ├── hosts.yml           # Server inventory
│   └── host_vars/          # Host-specific variables
├── roles/                   # Custom Ansible roles
│   ├── ansible_pull_setup/ # Ansible-pull automation
│   ├── chezmoi/            # Dotfiles management
│   ├── docker_compose_modules/  # Modular Docker Compose
│   └── ufw/                # Firewall configuration
└── scripts/
    └── ansible-pull.sh     # Installation wrapper

tests/
└── bash/                    # Bats test suite
    ├── run-tests.sh        # Test runner
    ├── lint-test.bats      # Linting tests
    ├── syntax-test.bats    # Syntax tests
    ├── docker-test.bats    # Docker provisioning tests
    ├── ansible-pull-test.bats  # ansible-pull tests
    └── github-ssh-keys-test.bats  # GitHub SSH keys tests
```

## Code Conventions

### Ansible Playbooks and Roles
- Use YAML with 2-space indentation
- Follow Ansible best practices for role structure
- All playbooks must start with `---`
- Use descriptive task names with proper capitalization
- Use `ansible.builtin.*` module names explicitly
- Always include `meta/main.yml` with role dependencies
- Use `defaults/main.yml` for default variables
- Use `handlers/main.yml` for service restarts

### YAML Style
- Use 2-space indentation (never tabs)
- No trailing whitespace
- End files with a single newline
- Use `---` document start marker
- Quote strings when they contain special characters
- Use lowercase for booleans: `true`, `false`

### Naming Conventions
- Roles: lowercase with underscores (e.g., `ansible_pull_setup`)
- Variables: lowercase with underscores (e.g., `server_features`)
- Host names: lowercase (e.g., `svlazext`)
- Tags: lowercase (e.g., `docker`, `traefik`)

### File Organization
- Group related tasks in role subdirectories
- Use `tasks/main.yml` as the entry point for roles
- Split complex roles into separate task files
- Store templates in `templates/` directory
- Keep defaults in `defaults/main.yml`

## Testing Requirements

### Before Making Changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevSecNinja/docker](https://github.com/DevSecNinja/docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
