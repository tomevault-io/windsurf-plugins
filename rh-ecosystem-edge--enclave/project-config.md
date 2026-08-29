---
trigger: always_on
description: Red Hat Sovereign Enclave (RHSE) is an optionally disconnected infrastructure platform that
---

## Project overview

Red Hat Sovereign Enclave (RHSE) is an optionally disconnected infrastructure platform that
delivers a cloud-like experience based on OpenShift. It provisions and maintains OpenShift clusters
on bare metal hardware, supports a local management plane (ACM, Quay), and controls software
ingress into air-gapped environments.

## Languages and tooling

- **Ansible** — deployment automation (playbooks/, 66+ playbooks in 7 phases)
- **Python 3.12** — reconciliation engine and tools (`src/`)
- **Bash** — provisioning and setup scripts (`scripts/`, 53+ scripts)
- **Jinja2** — cluster config templates (`templates/`)
- **YAML/JSON** — configuration and schema validation (`config/`, `schemas/`)

## Repository layout

```
playbooks/       Ansible playbooks: 01-prepare → 07-configure-discovery
plugins/         Optional components (lvms, odf, openshift-ai, nvidia-gpu, vast-csi)
experiences/     Experience bundles (collections of plugins, e.g. osac, aiaas)
src/             Python source root (src layout)
  reconcile/     Cluster/operator version reconciliation (enclave reconcile subcommand)
  tools/         Additional Python tools (enclave tools subcommand); new tools go here
  cli.py         Unified CLI entry point (reconcile + tools subcommands)
  utils.py       Shared utilities for all Python packages under src/
  tests/         All Python tests (pytest, shared fixtures)
scripts/         Shell scripts organized by function (setup, infrastructure, deployment, …)
templates/       Jinja2 templates for cluster and registry configs
schemas/         JSON schemas for config and plugin descriptor validation
config/          User-provided cluster configuration (gitignored, examples provided)
defaults/        Default variable values (catalogs, operators, platforms)
docs/            Deployment, configuration, and architecture guides
scripts/docs/    CI-related documentation (runner setup, workflow guides, troubleshooting)
```

## Building and testing

```bash
# Python
make python-unit-test     # pytest with coverage required (see pyproject.toml)
make python-linter-test   # ruff formatting check
make python-types-test    # mypy strict type checking
make python-format        # auto-format Python code

# Infrastructure validation (runs on every PR)
make -f Makefile.ci validate             # all checks
make -f Makefile.ci validate-shell       # shellcheck
make -f Makefile.ci validate-yaml        # yamllint
make -f Makefile.ci validate-ansible     # ansible-lint
make -f Makefile.ci validate-plugins     # plugin descriptor validation
```

## Code conventions

### Python (`src/`)
- Strict mypy: all functions must have type annotations
- ruff: 88-char line limit, comprehensive linting and import sorting
- Custom exception hierarchy with descriptive messages
- Click-based CLIs with subcommands (one per package)
- Shared utilities live in `src/utils.py` (includes `configure_logging()` for CLI entry points); new Python tools go under `src/tools/`
- Check python-*-test Makefile targets

### Ansible (`playbooks/`)
- Phase-based structure: numbered playbooks orchestrate reusable tasks from `playbooks/tasks/`
- Tasks must be idempotent and re-runnable
- Use descriptive `name:` fields on all tasks
- Explicit `become: yes` for privilege escalation

### Shell (`scripts/`)
- `set -euo pipefail` at the top of every script
- Source shared utilities from `scripts/lib/` (logging, env checks, etc.)

### Plugins
- Each plugin has a single `plugin.yaml` descriptor validated against `schemas/plugin.yaml`
- Optional lifecycle task files: `tasks/early-validate.yaml`, `tasks/deploy.yaml`, `tasks/post-validate.yaml`
- Declarative operator and registry requirements in the descriptor

### Config and schemas (`config/`, `defaults/`, `schemas/`)

- Every property added to a file under `config/` or `defaults/` (including plugin example/default
  configs) must also be added to its matching schema file under `schemas/` (same base filename,
  e.g. `defaults/catalogs.yaml` → `schemas/catalogs.yaml`) in the same PR
- Run `make -f Makefile.ci validate-json-schema` before opening a PR to catch missing schema entries

## Git workflow

- **NEVER push commits directly to `main`** — all changes must go through pull requests
- This project does not use forks — create feature branches in the main repository
- Branch naming: use descriptive names like `feature/add-xyz`, `fix/bug-description`, `docs/update-readme`
- When work is ready, create a PR for review — do not push to main even if you have permissions

### GitHub CLI (Recommended)

Use the [gh](https://cli.github.com/) tool for efficient GitHub workflow management from the command line.

**Installation** (choose the method that fits your environment):
```bash
brew install gh              # macOS/Linux
sudo dnf install gh          # Fedora/RHEL
sudo apt install gh          # Debian/Ubuntu
# Immutable distros: use toolbox/distrobox or download binary
# Direct download: https://github.com/cli/cli/releases
```

**Common operations**:
```bash
gh auth login  # Initial authentication
gh pr create --title "OSAC-123: Add feature" --body "..."
gh pr view 432
gh pr comment 432 --body "✨ **Claude Code**: Fixed in abc1234"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rh-ecosystem-edge/enclave](https://github.com/rh-ecosystem-edge/enclave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
