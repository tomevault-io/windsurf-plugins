---
trigger: always_on
description: Ensure that all practices and instructions described by
---

# Ansible Collection: artis3n.tailscale - AI Agent Instructions

Ensure that all practices and instructions described by
https://raw.githubusercontent.com/ansible/ansible-creator/refs/heads/main/docs/agents.md
are followed.

## Project Overview

This is an Ansible Collection for installing and managing Tailscale VPN nodes on Linux systems. The collection follows Ansible Collection's canonical directory structure and is published to Ansible Galaxy. The project contains the `artis3n.tailscale.machine` role for Tailscale client management.

## Architecture

### Collection Structure

- **Role**: [`roles/machine/`](../roles/machine/) - Main role for Tailscale installation/configuration
  - Multi-distro support (Debian/Ubuntu, CentOS/RHEL/Rocky/Alma, Fedora, Arch, OpenSUSE)
  - OS-specific installation logic in [`tasks/<distro>/install.yml`](../roles/machine/tasks/)
  - State management via idempotency checks using hashed state files (`~/.local/state/artis3n-tailscale/state`)
- **Test Suite**: [`extensions/molecule/`](../extensions/molecule/) - Molecule test scenarios for different use cases
- **Dependencies**: Requires `community.general` collection (defined in [`galaxy.yml`](../galaxy.yml))

### Key Design Patterns

#### Multi-Distribution Support Pattern
The role uses conditional includes based on `ansible_facts.distribution` to route to OS-specific installation tasks:
- Distribution families defined in [`roles/machine/vars/main.yml`](../roles/machine/vars/main.yml) (e.g., `tailscale_debian_family_distros`)
- Each distro has its own install/uninstall task files in [`roles/machine/tasks/<distro>/`](../roles/machine/tasks/)
- Package repository URLs are templated with `release_stability` variable (stable/unstable)

#### State Idempotency Pattern
The role tracks `tailscale up` command state to avoid unnecessary re-authentication:
- State file stored in [`~/.local/state/artis3n-tailscale/state`](../roles/machine/tasks/files/state_readme.md) (XDG Base Directory spec)
- Contains SHA256 hash of `tailscale_args` + authkey (see [`tasks/templates/state.j2`](../roles/machine/tasks/templates/state.j2))
- `tailscale up` only runs when state changes or Tailscale is offline

#### OAuth vs. Auth Key Handling
The role supports two Tailscale authentication methods:
- **OAuth keys** (`tskey-client-*`): Require `tailscale_tags`, support `tailscale_oauth_ephemeral` and `tailscale_oauth_preauthorized`
  - Authkey string is modified to include OAuth parameters: `<key>?ephemeral=<bool>&preauthorized=<bool>`
- **Auth keys** (`tskey-auth-*`): Traditional node authorization keys
- Detection and validation logic in [`tasks/install.yml`](../roles/machine/tasks/install.yml)

#### Fact Gathering Pattern
After successful `tailscale up`, the role registers facts about the node:
- Queries `tailscale ip --4/--6` and `tailscale whois --json` (see [`tasks/facts.yml`](../roles/machine/tasks/facts.yml))
- Exports: `tailscale_node_ipv4`, `tailscale_node_ipv6`, `tailscale_node_hostname_*`, `tailscale_node_tags`, etc.

## Development Workflow

### Environment Setup
```bash
make install  # Uses uv to sync Python deps, install pre-commit hooks, and install Ansible collections
```
- **Python environment**: Uses `uv` (see [`pyproject.toml`](../pyproject.toml)) with Python 3.13
- **Ansible dependencies**: Installed via `ansible-galaxy install -r requirements.yml` to `.ansible-dependencies/`
- **Pre-commit hooks**: Automatically configured (includes ansible-lint)

### Testing with Molecule

Tests are run from the [`extensions/`](../extensions/) directory. This follows Molecule's best practices for testing Ansible Collections. The working directory path `collections/ansible_collections/artis3n/tailscale` must be maintained for collection resolution. This requirement ensures that Ansible can properly locate the collection and its dependencies.

#### Test Environment Variables
- `TAILSCALE_CI_KEY`: Required ephemeral auth key for most scenarios
- `TAILSCALE_OAUTH_CLIENT_SECRET`: Required for OAuth scenario testing
- `USE_HEADSCALE=true`: Use local Headscale container instead of Tailscale cloud (optional)
- `MOLECULE_DISTRO`: Override test container image (e.g., `geerlingguy/docker-ubuntu2404-ansible:latest`)

#### Running Tests
```bash
# From project root:
make test        # Runs default + state-absent scenarios
make testd       # Runs role-machine-default scenario only

# From roles/machine/:
make test-default      # Default scenario
make test-oauth        # OAuth authentication scenario
make test-headscale    # Using local Headscale
```

#### Molecule Scenario Structure
Each scenario in [`extensions/molecule/`](../extensions/molecule/) contains:
- `molecule.yml`: Defines Docker containers (instance + optional headscale), sets `ANSIBLE_COLLECTIONS_PATH`
- `converge.yml`: Playbook that applies the role
- `verify.yml`: Assertions to validate the scenario
- `cleanup.yml`: De-registration tasks (some scenarios)

The `ANSIBLE_COLLECTIONS_PATH` environment variable is critical - it's set to `"${MOLECULE_PROJECT_DIRECTORY}/../../../..:${MOLECULE_PROJECT_DIRECTORY}/../.ansible-dependencies"` to resolve the collection and its dependencies.

### CI/CD Pipeline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artis3n/ansible-collection-tailscale](https://github.com/artis3n/ansible-collection-tailscale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
