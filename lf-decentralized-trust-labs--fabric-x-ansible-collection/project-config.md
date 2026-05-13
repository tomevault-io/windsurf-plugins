---
trigger: always_on
description: `hyperledger.fabricx` is an Ansible collection that automates deployment and lifecycle management of **Hyperledger Fabric-X** networks.
---

# AGENTS.md — AI Agent Guide for `hyperledger.fabricx`

`hyperledger.fabricx` is an Ansible collection that automates deployment and lifecycle management of **Hyperledger Fabric-X** networks.
Namespace/name: `hyperledger.fabricx`. Authoritative version and deps: [`galaxy.yml`](galaxy.yml).

---

## Rules — follow before every commit

1. **License header** — every YAML, shell, and Jinja2 file must begin with:

   ```yaml
   #
   # Copyright IBM Corp. All Rights Reserved.
   #
   # SPDX-License-Identifier: Apache-2.0
   #
   ```

   CI enforces this via `scripts/check_license_header.sh`.

2. **No trailing spaces** in `.j2` files — enforced by `scripts/check_trailing_spaces.sh`.
3. **Idempotency** — all tasks must be idempotent; use `creates:`, `changed_when:`, or appropriate modules.
4. **Task names** — every `ansible.builtin.*` task must have a `name:` field.
5. **Code order** — first `name:` field, then `vars:` (if needed), FQDN name of task and finally `when:` (if needed). For blocks: `name:`, then `when:`, then `block:`.
6. **Templates** — Jinja2 templates go in `roles/<role>/templates/` with `.j2` extension.

---

## Working in isolation

Use git worktrees under `.worktrees/` only for multi-agent jobs (i.e. with subagents). For single-agent work, edit the main checkout directly.

```shell
git worktree add .worktrees/<3-4-word-slug> -b worktree/<3-4-word-slug>
```

Do **not** remove the worktree when done — leave cleanup to the user.

---

## Architecture

### Always read the role argument_specs.yaml first

Before modifying any role, read `roles/<role>/meta/argument_specs.yaml`. It is the authoritative reference for that role: available tasks, variables, and which deployment modes (binary / container / k8s) it supports. Deployment mode support varies per role — do not assume.

### Dispatch pattern

Roles that manage multiple sub-components (e.g. `orderer`: consenter/batcher/assembler/router; `committer`: validator/verifier/coordinator/sidecar/query-service) use a dispatcher: the top-level task file reads `<role>_component_type` and delegates to the matching sub-component directory:

```yaml
ansible.builtin.include_role:
  name: hyperledger.fabricx.<role>
  tasks_from: <sub_component>/start # e.g. coordinator/start, assembler/bin/install
```

### Role layout

```text
roles/<role>/
├── defaults/main.yaml          # auto-generated from meta/argument_specs.yaml
├── meta/argument_specs.yaml    # single source of truth for variables and docs
├── tasks/
│   ├── start.yaml              # top-level dispatcher (reads *_component_type)
│   ├── <sub_component>/
│   │   ├── bin/
│   │   ├── container/
│   │   └── k8s/
└── templates/                  # *.j2 Jinja2 templates
```

---

## Cross-role dependencies

These connections are not visible from within a single role:

| Dependency                                         | Detail                                                                                                                                                                                                          |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `committer` → `postgres`                           | Started when `postgres_port` is defined in inventory                                                                                                                                                            |
| `committer` → `yugabyte`                           | Started when `yugabyte_component_type` is defined in inventory                                                                                                                                                  |
| `committer` ↔ `orderer`                            | Coordinator receives the assembler host list at startup                                                                                                                                                         |
| `fxconfig` → `committer`, `orderer`                | Generates configs consumed by both; for k8s deployments also runs namespace creation                                                                                                                            |
| `cryptogen` / `fabric_ca` → `orderer`, `committer` | Crypto artifacts must exist before either can be configured or started                                                                                                                                          |
| `armageddon` / `configtxgen` → crypto              | Genesis block generation depends on crypto output                                                                                                                                                               |
| `k8s` role → k8s deployments                       | Namespace setup is a prerequisite for any k8s-mode deployment                                                                                                                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LF-Decentralized-Trust-labs/fabric-x-ansible-collection](https://github.com/LF-Decentralized-Trust-labs/fabric-x-ansible-collection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
