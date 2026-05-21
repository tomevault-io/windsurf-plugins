---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Ansible playbook for a self-hosted infrastructure on Ubuntu ARM64 VPS.

## Server Access — ALWAYS Use Ansible (MOST IMPORTANT RULE)

**NEVER use raw `ssh` to run commands on the server. ALWAYS go through Ansible.** Use ad-hoc commands for one-off diagnostics:

```bash
# Run a shell command on the server (becomes root automatically)
ansible server -i inventory.yml -e @custom.yml -e @secret.yml \
  --vault-password-file ~/.vault_pass \
  -m ansible.builtin.shell -a 'docker ps' -b

# Use community.docker.docker_container_exec to run inside a container
ansible server -i inventory.yml -e @custom.yml -e @secret.yml \
  --vault-password-file ~/.vault_pass \
  -m community.docker.docker_container_exec \
  -a 'container=amnezia-wg command="awg show"' -b
```

Why: Ansible enforces consistent become/sudo handling, vault decryption, inventory-driven host resolution, and produces structured/idempotent output. Raw SSH bypasses all of that and creates state drift.

## Security Rules

- **NEVER** read `secret.yml` or `~/.vault_pass` — these contain sensitive credentials and must not be sent to the cloud.
- If playbook output contains secret values, do not display them.
- The vault password must never appear in conversation.

## Security Model

Defense-in-depth is split across two layers that must be kept in sync:

- **Hetzner cloud firewall (external layer, managed in Hetzner console, NOT in this repo):**
  - SSH (`22/tcp`) is IP-allowlisted here — only specific operator IPs can reach the server.
  - This is the authoritative SSH restriction. If the firewall is removed, disabled, or the allowlist is cleared, the server is exposed.

- **Host iptables (internal layer, managed by `roles/system/templates/iptables.conf`):**
  - Intentionally permissive for SSH (`ssh_allow_cidr: 0.0.0.0/0` in `inventory.yml`) because the Hetzner firewall already gates it.
  - Treat this as a fail-open second layer, not a primary defense.

When changing SSH exposure, update **both** layers. If the Hetzner firewall is ever removed, tighten `ssh_allow_cidr` in `inventory.yml` to a real CIDR before redeploying.

### IPv6

IPv6 is disabled at the Hetzner cloud provider layer — the server's Primary
IPv6 has been detached in the console, and all IPv6 sources have been
removed from the cloud firewall rules. As a result:

- No IPv6 traffic is routed to or from the server by Hetzner.
- The VM's OS may still show a stale `inet6 2a01:…` address on `eth0` and a
  v6 link-local default route until the next cloud-init regeneration or
  reboot; this is cosmetic and **not** reachable from the internet.
- Deliberately there is **no `ip6tables` / `ip6tables.conf`** in this repo.
  The `iptables.conf` template covers v4 only. If IPv6 is ever re-attached
  in the Hetzner console, mirror the allowed-ports list to a new
  `ip6tables.conf` drop-in and review `sshd` `AddressFamily` before
  redeploying.

## Running the Playbook

Install dependencies first (once, or when `requirements.yml` changes):

```bash
ansible-galaxy install -r requirements.yml
```

Full deploy:

```bash
ansible-playbook playbook.yml -i inventory.yml -e @custom.yml -e @secret.yml --vault-password-file ~/.vault_pass
```

Deploy specific role(s) by tag:

```bash
ansible-playbook playbook.yml -i inventory.yml -e @custom.yml -e @secret.yml --vault-password-file ~/.vault_pass --tags "traefik,authelia"
```

Lint the project:

```bash
ansible-lint playbook.yml
yamllint .
```

Dry-run (check mode):

```bash
ansible-playbook playbook.yml -i inventory.yml -e @custom.yml -e @secret.yml --vault-password-file ~/.vault_pass --check --tags "<tag>"
```

## Configuration

Before running, two files must exist (both gitignored):

- **`custom.yml`** — copy from `.custom.yml`, fill in `ansible_host`, `root_host`, `username`, SSH key path, email settings, `force_recreate`, and storage config.
- **`secret.yml`** — copy from `.secret.yml`, fill in all secrets. Must be `chmod 600` and encrypted with `ansible-vault`.

`ansible.cfg` sets `roles_path = .ansible/roles`, `collections_path = .ansible/collections`, enables `profile_tasks` callback, YAML output format, SSH connection multiplexing, and disables retry files.

`inventory.yml` derives all service subdomains from `root_host` (e.g., `auth.{{ root_host }}`, `password.{{ root_host }}`). Docker data lives under `docker_dir` (default `/opt/docker`). Storage box mounts go under `storagebox_mount_path` (default `/mnt/storagebox`).

## Architecture

### Role Execution Order

Roles in `playbook.yml` run in this sequence and must be thought of as layers:

1. **Infrastructure base**: `system` → `security` → `dns` → `geerlingguy.docker` → `docker_network` → `docker_proxy`
2. **Security/proxy layer**: `crowdsec` → `traefik` → `redis` → `authelia`
3. **Applications**: all remaining roles (each has its own tag matching its folder name)

External roles (installed to `.ansible/roles/`):

- `geerlingguy.docker`
- `geerlingguy.pip`
- `chriswayg.msmtp-mailer`

External collections (installed to `.ansible/collections/`):

- `community.docker` (>=5.0.0)
- `community.general`
- `community.crypto`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atomdeniz/server](https://github.com/atomdeniz/server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
