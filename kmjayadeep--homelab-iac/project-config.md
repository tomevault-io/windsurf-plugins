---
trigger: always_on
description: Applies to the entire repository unless a deeper `AGENTS.md` exists.
---

# AGENTS.md

## Scope
Applies to the entire repository unless a deeper `AGENTS.md` exists.

## Project Overview
Infrastructure-as-code repo with:
- Terraform stacks (`cloudflare/`, `hetzner/terraform/`, `cosmos/proxmox/`, `cosmos/pluto/`, `storage/minio/`, `vault-config/`).
- Terraform modules (`terraform-modules/*`).
- Ansible projects (`cosmos/ansible-k3s/`, `cosmos/ansible-valheim/`, `hetzner/ansible-nova/`).
- NixOS flake images under `nixos-images/*`.
- Shell scripts and Justfiles for infra workflows.

## Commands: Build / Lint / Test
No dedicated lint/test frameworks were found. Use the scoped commands below to
validate or apply infra changes per component.

### Terraform (per directory)
Run these in the specific Terraform root directory.
- `terraform init`
- `terraform plan`
- `terraform apply`

Terraform directories include:
- `cloudflare/`
- `hetzner/terraform/`
- `cosmos/proxmox/`
- `cosmos/pluto/`
- `vault-config/`
- `storage/minio/`
- `terraform-modules/*` (modules only; no direct apply)

Single target plan/apply (single "test" equivalent):
- `terraform plan -target=resource.type.name`
- `terraform apply -target=resource.type.name`

Format/validate (if needed):
- `terraform fmt -recursive`
- `terraform validate`

### Terraform (Justfile helpers)
Some stacks provide `just` recipes.
- `cosmos/pluto/justfile`
  - `just plan`
  - `just apply`
- `cosmos/proxmox/justfile`
  - `just plan`
  - `just apply`

### Ansible (K3s master)
From `cosmos/ansible-k3s/`:
- Install collections: `ansible-galaxy collection install -r requirements.yml`
- Connectivity: `ansible all -m ping`
- Deploy master: `ansible-playbook playbooks/setup.yml`
- Start/stop/restart/update:
  - `ansible-playbook playbooks/start.yml`
  - `ansible-playbook playbooks/stop.yml`
  - `ansible-playbook playbooks/restart.yml`
  - `ansible-playbook playbooks/update.yml`

Worker nodes (single host test):
- Deploy: `ansible-playbook playbooks/setup-workers.yml`
- Start/stop:
  - `ansible-playbook playbooks/start-workers.yml`
  - `ansible-playbook playbooks/stop-workers.yml`
- Single host limit: `ansible-playbook playbooks/setup-workers.yml -l host_or_group`

### Ansible (Valheim)
From `cosmos/ansible-valheim/`:
- Install collections: `ansible-galaxy collection install -r requirements.yml`
- Connectivity: `ansible all -m ping`
- Deploy: `ansible-playbook playbooks/setup.yml`
- Start/stop/restart/update:
  - `ansible-playbook playbooks/start.yml`
  - `ansible-playbook playbooks/stop.yml`
  - `ansible-playbook playbooks/restart.yml`
  - `ansible-playbook playbooks/update.yml`
- Single host limit: `ansible-playbook playbooks/setup.yml -l host_or_group`

### Ansible (Nova)
From `hetzner/ansible-nova/`:
- Deploy: `./deploy.sh`
- Update containers: `./update.sh`
- Manual playbook: `ansible-playbook playbooks/setup.yml`
- Single host limit: `ansible-playbook playbooks/setup.yml -l host_or_group`

### NixOS images
From `nixos-images/*` directories, use the Makefiles to rebuild.
Examples:
- `nixos-images/k3s/`: `make rebuild`
- `nixos-images/valheim/`: `make rebuild-fireland`
- `nixos-images/postgres/`: `make rebuild`
- `nixos-images/gatekeeper/`: `make rebuild`
- `nixos-images/github-runner/`: `make rebuild`
- `nixos-images/jd-workspace/`: `make rebuild`
- `nixos-images/golem/`: `make rebuild`
- `nixos-images/coder/`: `make rebuild`

Base image build (from `nixos-images/nixos-base-image/`):
- `nix build .#proxmox`

### Shell scripts
Run from the directory the script is in.
- Use `bash -n script.sh` for a quick syntax check.

### Misc
No repo-wide build/test commands discovered.

## Code Style Guidelines
Follow existing patterns in each subdirectory. Keep changes small and
consistent with the surrounding files.

### General
- Prefer minimal, targeted edits.
- Avoid introducing new tooling or lint rules without approval.
- Do not commit plaintext credentials. Use `.envrc`, `pass`, and environment variables for provider authentication.
- Keep comments concise and only when they add clarity.
- Respect `.gitignore` and never add `.tfvars` or credential files. The existing Vault Terraform state is an explicit exception: it is tracked through `git-crypt` and must remain encrypted.
- Keep files organized under existing directories; avoid renames unless needed.

### Vault and External Secrets
- Read `vault-config/PATHS.md`, `vault-config/README.md`, and `plans/0001-vault-secret-migration.md` before changing Vault paths, policies, or Kubernetes roles.
- Terraform manages Vault mounts, auth methods, exact-path policies, and roles; it must not manage secret values.
- Write or rotate values through a non-logging stdin/file workflow. Never put values in Terraform, command arguments, shell history, output, plans, or agent messages.
- KV paths use the owner-based `apps/`, `services/`, and `platform/` taxonomy. Platform consumers share a canonical path through separate policies rather than consumer-specific copies.
- Custom metadata is non-secret and follows `vault-config/PATHS.md`. Keep `origin`, `owner`, management, and lifecycle fields current.
- `external_secrets_roles` must grant exact logical paths and bind a dedicated Kubernetes ServiceAccount and namespace.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kmjayadeep/homelab-iac](https://github.com/kmjayadeep/homelab-iac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
