---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Agent Safety Rules

- This repository is the source of truth. Make changes in Git, validate locally, and let Flux reconcile them.
- Do not run mutating `kubectl` commands unless the user explicitly authorizes the exact action first.
- Forbidden without prior authorization: `kubectl apply`, `create`, `delete`, `replace`, `patch`, `edit`, `scale`, `rollout restart`, `annotate`, `label`, `cordon`, `drain`, and any other command that changes live cluster state.
- Read-only inspection is allowed: `kubectl get`, `describe`, `logs`, `events`, `top`, `auth can-i`, and `diff` or `apply --dry-run=server`.
- Prefer the Flux MCP server, `flux` read commands, and local rendering/diff tools for troubleshooting. If a live change is necessary, stop and ask first with the exact command and reason.
- `flux reconcile ...` is allowed only to ask Flux to apply committed Git state or when explicitly requested by the user; do not use Flux as a substitute for direct manifest application.

## Cluster

This is **kantai**, a Kubernetes cluster running Talos Linux, with a mix of bare-metal and virtual nodes, managed entirely through GitOps via FluxCD and Flux Operator. All services live under `kantai.xyz`. Current repo configuration targets Talos `v1.13.2` and Kubernetes `v1.36.1`. Flux syncs `refs/heads/main` from `https://github.com/jfroy/flatops` at `kubernetes/cluster`.

## Flux MCP Server

A Flux MCP server may be configured. Always use the `kantai.hyakutake-universe.ts.net` kubeconfig context. Use it to inspect live cluster state when troubleshooting — check HelmRelease/Kustomization status, events, and inventory before editing files.

## Maintenance Commands

```sh
# Talos: regenerate node configs from talconfig.yaml and apply
task talos:gen-mc
task talos:apply-mc

# Apply to a single node
task talos:apply-node HOSTNAME=kantai1
```

Flux reconciliation, when appropriate and after Git state is ready:

```sh
flux reconcile kustomization cluster-apps --with-source
flux reconcile helmrelease <name> -n <namespace>
```

Do not use `kubectl apply -k`, `kubectl apply -f`, or direct Helm installs/upgrades to deploy repo resources unless the user has explicitly authorized that live mutation.

## Repository Structure

```txt
kubernetes/
  apps/             # One subdirectory per namespace
    default/        # Most user-facing applications
  cluster/          # Root Flux entrypoint (cluster-vap -> cluster-apps)
  components/       # Reusable Kustomize components
    common/         # Adds NS-level alerts + ExternalSecret + prune annotation
    volsync/        # Adds VolSync ReplicationSource + PVC to an app
    volsync-ns/     # Adds VolSync ExternalSecret at namespace level
  transformers/     # NamespaceTransformer applied globally
  vap/              # ValidatingAdmissionPolicies applied before apps
talos/              # talhelper config (talconfig.yaml + SOPS-encrypted secrets)
bootstrap/          # One-time cluster bootstrap (currently broken/unused)
```

## Helm Chart Strategy

Two categories of deployments exist in this cluster:

- **app-template apps** — containerized applications without their own Helm chart. These use the `bjw-s-labs/app-template` chart, which provides a generic, highly-configurable template for deploying arbitrary containers. This covers most user-facing apps under `kubernetes/apps/default/`.
- **official-chart apps** — cloud-native projects and infrastructure components that ship their own Helm chart (e.g. cert-manager, external-secrets, cilium, CNPG, Flux itself). Always prefer the upstream chart for these; only fall back to app-template if the official chart has a serious problem.

OCIRepository sources are strongly preferred over `HelmRepository` (HTTP/HTTPS) sources. When an upstream chart is not yet available as an OCI artifact, check the [home-operations/charts-mirror](https://github.com/home-operations/charts-mirror) community registry first — it mirrors many popular charts as OCI images at `ghcr.io/home-operations/charts-mirror`.

## App Pattern (kubernetes/apps/default/)

App-template apps follow the same four-file layout:

```txt
<appname>/
  ks.yaml               # Flux Kustomization — registers the app with Flux
  app/
    helmrelease.yaml    # HelmRelease — app-template or upstream chart via OCIRepository/HelmRepository
    kustomization.yaml  # Kustomize manifest listing resources in app/
    externalsecret.yaml # Secrets pulled from 1Password via external-secrets
```

**`ks.yaml` key points:**

- References `components/volsync` to wire up daily Kopia backups to Cloudflare R2
- Uses `postBuild.substitute` with `APP: *app` — this variable names the PVC and backup target
- Optionally `VOLSYNC_CAPACITY` (PVC size) and `APP_SUBDOMAIN` (if subdomain ≠ app name)
- Postgres apps add `dependsOn: cnpg-pg18vc` in `cnpg-system`

**`helmrelease.yaml` key points:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jfroy/flatops](https://github.com/jfroy/flatops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
