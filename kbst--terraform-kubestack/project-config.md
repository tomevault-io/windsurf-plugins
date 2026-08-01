---
trigger: always_on
description: > This file governs using the Kubestack framework to define and manage Kubernetes platform infrastructure.
---

# Kubestack AGENTS.md — Framework Usage

> This file governs using the Kubestack framework to define and manage Kubernetes platform infrastructure.
> For contributing to the framework itself, see the `AGENTS.md` in the framework repository root.

---

## Start Here

**Before doing anything else, read the Platform section of `README.md`.**
Every value already recorded there must be used as-is — never ask the user for a value that is already recorded.
If a required value is missing, ask the user, record the answer in `README.md`, then write the files.

---

## Constraints

These rules are absolute. They apply to every task without exception.

### Commands you must never run

- `terraform apply` / `tofu apply`
- `terraform destroy` / `tofu destroy`
- `terraform init` / `tofu init`
- `terraform plan` / `tofu plan`
- Any `git` command that's not read-only — no commits, pushes, branch creation, or tagging

These commands make irreversible changes to live infrastructure or the repository state. They must always be run by a human who has reviewed what will happen. If the user asks you to run any of them, explain that you cannot do so and give them the exact command to run themselves.

### The one command you must run

After writing or editing any `.tf` file, run `tofu validate` (or `terraform validate`) to verify syntax. This command is safe — it makes no cloud API calls and changes nothing.

### Other hard rules

- Never modify `.terraform.lock.hcl` by hand.
- Never hardcode secrets, credentials, keys, or certificates in any `.tf` file.
- Never edit `modules/<feature_name>/manifests/upstream.yaml` directly for Helm-based features. It is a generated file. To change what is rendered, edit `values.yaml` and re-run the `helm template` command recorded in `modules/<feature_name>/README.md` to regenerate it.
- Always update the Platform section of `README.md` after any task that adds, changes, or removes a platform component or setting.
- Always give the user the GitOps follow-up instructions at the end of every task. See [After every task](#after-every-task).

---

## Update Obligations

Before considering any task complete, apply every obligation from this table that matches what was done.

| What was done | Obligation |
|---|---|
| First-time repository scaffold | Fill in the Configuration table in `README.md` (framework version, environment names, base environment, `base_domain`). Add the first cluster to the Clusters table. |
| New cluster scaffolded (Stage 1) | Add a row to the Clusters table in `README.md`. Give the user the GitOps follow-up instructions and remind them that node pools and platform features must be added in separate subsequent stages. |
| Cluster removed | Remove its row from the Clusters table in `README.md`. Remove all its node pool and feature rows too. |
| New node pool scaffolded | Add a row to the Node Pools table in `README.md`. |
| Node pool removed | Remove its row from the Node Pools table in `README.md`. |
| New platform feature scaffolded | Add a row to the Platform Features table in `README.md`. |
| Platform feature removed | Remove its row from the Platform Features table in `README.md`. |
| Framework version changed | Update the framework version in the Configuration table in `README.md`. Update the `?ref=` value in every `source` reference to a Kubestack framework module across the entire repository. Update the Kubestack image tag in `Dockerfile` to the same version. |
| Environment added | Add the new environment name to the Configuration table in `README.md`. Add the new key to the `configuration` map in every module in the repository. |
| Environment removed | **Destructive.** Do not make any file changes immediately. Follow the staged process in [Removing an environment](#removing-an-environment). |
| Environment renamed | **Destructive.** Do not make any file changes immediately. Follow the staged process in [Renaming an environment](#renaming-an-environment). |
| Any task completed | Give the user the full GitOps follow-up instructions from [After every task](#after-every-task). |

When updating `README.md`: replace existing values. Never append duplicates or leave stale entries alongside new ones.

---

## Framework Version

The Kubestack framework version governs all release assets in the repository. These must always be identical — upstream releases bundle modules and the OCI image together so that all assets stay aligned:

- The `?ref=<version>` parameter in every Kubestack framework module `source` reference
- The container image tag in `Dockerfile`

Read the current version from the Configuration table in `README.md`.
When adding any new module to an existing repository, always match the version already in use.
When initialising a brand-new repository or updating the framework version, fetch the latest version from: https://www.kubestack.com/cli.json

All framework module sources follow the same GitHub format:

```
# cluster module
source = "github.com/kbst/terraform-kubestack//aws/cluster?ref=v0.18.1-beta.0"

# node-pool module
source = "github.com/kbst/terraform-kubestack//aws/cluster/node-pool?ref=v0.18.1-beta.0"

# platform feature module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kbst/terraform-kubestack](https://github.com/kbst/terraform-kubestack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
