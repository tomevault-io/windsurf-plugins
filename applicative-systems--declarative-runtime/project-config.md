---
trigger: always_on
description: > Status: the pattern is implemented. **`services/forgejo` is the worked
---

# CLAUDE.md

> Status: the pattern is implemented. **`services/forgejo` is the worked
> reference pairing** — new pairings are modeled on it, and the "Provider
> implementation contract" below is exactly what it encodes. Grafana and
> Keycloak (see "Target pairings") are designed but not yet built; do not
> present them as implemented.

## Purpose

Make NixOS services **more declaratively configurable** than upstream Nixpkgs
modules allow, by pairing each service with its Terraform provider and
reconciling the service's _runtime state_ once the service is up.

Upstream NixOS modules configure a service's **static** surface — package
version, config file, the systemd unit. They deliberately do **not** manage a
service's **runtime** state: Grafana dashboards/datasources, a Git forge's
orgs/repos/teams, etc. Many such services ship a Terraform provider that _does_
manage exactly that state.

This repo closes the gap: you declare the desired runtime state in Nix, and a
systemd unit applies it (via OpenTofu) against the live, local instance after
the service's primary unit starts.

A pairing only makes sense when the service has **admin-declarative runtime
state reachable through a provider** that the NixOS module cannot already
express. Services whose entire surface is config-file-driven (and thus already
declarative via their NixOS options) are out of scope. (Authelia is the
canonical _non_-fit: no Terraform provider exists, and its admin surface is
already covered by `services.authelia.*` — so it was dropped as a pairing.)

## Settled decisions

| Topic            | Decision                                                                                                                                                                                                                                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Executor         | **OpenTofu** (nixpkgs `opentofu`, MPL 2.0 / free). `terraform` is BSL 1.1 / unfree and is **not** used.                                                                                                                                                                                                    |
| Config authoring | Generate **`.tf.json`** directly from Nix (`builtins.toJSON`). No HCL, no terranix dependency.                                                                                                                                                                                                             |
| Secrets          | **systemd `LoadCredential=`** is the blessed path. Generated config references the secret as a `sensitive` Terraform variable; never literal secrets. sops-nix/agenix, if used, only supply the source that feeds `LoadCredential=`.                                                                       |
| Reconciliation   | **Run-once**: a `Type=oneshot` unit ordered `After=` the primary unit + readiness probe, runs `init` + `apply -auto-approve`. Re-applies on config change via `restartTriggers`. **No** drift timer. A failed apply fails _that unit_ visibly (`systemctl status`) and does **not** tear down the service. |
| State            | **Local, per-host only.** Terraform state lives under the **base service's primary state directory** (e.g. `services.forgejo.stateDir` → `/var/lib/forgejo`), co-located with the service it configures. No remote backends.                                                                               |
| Module namespace | Under the base service as **`services.<svc>.runtime.*`** (e.g. `services.forgejo.runtime.repositories`), so the pairing reads as a transparent extension of the upstream `services.<svc>` module.                                                                                                          |
| Formatter        | **treefmt** driving **nixfmt**. Formatter is the single source of layout truth — run it, never hand-format.                                                                                                                                                                                                |
| CI               | **GitHub Actions**: `nix flake check` on push + PR, Nix provided by Determinate Systems `nix-installer-action`. Workflow is Forgejo-Actions-compatible (same syntax) if hosting moves there.                                                                                                               |
| License          | **MIT** (matches nixpkgs ecosystem norms; permissive).                                                                                                                                                                                                                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [applicative-systems/declarative-runtime](https://github.com/applicative-systems/declarative-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
