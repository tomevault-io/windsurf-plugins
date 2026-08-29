---
trigger: always_on
description: Red Hat deployment artifacts for NVIDIA NICo (Infra Controller), part of the
---

# ncp-dsx-nico-deploy — AI Assistant Guide

## What This Is

Red Hat deployment artifacts for NVIDIA NICo (Infra Controller), part of the
DSX OS platform, targeting NVIDIA Cloud Partners (NCP). This repo does NOT
fork upstream — it installs upstream Helm charts directly with values overrides
and provides a thin Red Hat infrastructure layer (Crunchy PostgreSQL, RHBK
Keycloak, Vault HA, ESO, OpenShift Routes).

## Upstream Repos (read-only, never modified)

| Repo | What we consume |
|---|---|
| `github.com/NVIDIA/infra-controller` | Go/Rust source (monorepo: REST API, site-agent, Core) |

Vendored as git submodule at `helm/vendor/infra-controller`.

## Architecture

### Design Principle

Upstream charts are installed directly — never wrapped or re-packaged. Our
downstream layer is cleanly separated into four concerns:

1. **Values overrides** (`helm/values/`) — config deltas for upstream charts
2. **Infrastructure charts** (`helm/infra-cloud/`, `helm/infra-site/`) — Red Hat
   components (Crunchy PG, RHBK, Vault HA, ESO, Routes)
3. **Kustomize patches** (`helm/kustomize/`) — fixes for upstream templates
   (each patch maps to one upstream contribution PR)
4. **Prereqs** (`helm/nvidia-infra-controller-prereqs/`) — OLM operator subscriptions

### Deployment Topology

```
nico-rest namespace (management plane — upstream nico-rest chart)
├── nico-rest-api                    ← upstream
├── nico-rest-cloud-worker           ← upstream
├── nico-rest-site-worker            ← upstream
├── nico-rest-site-manager           ← upstream
├── nico-rest-cert-manager           ← upstream
├── nico-rest-db (migration job)     ← upstream
├── Temporal server v1.2.0           ← upstream chart from go.temporal.io
├── nico-cloud-pg (Crunchy PG18)     ← infra-cloud (nico, temporal, keycloak DBs)
├── OpenShift Routes                 ← infra-cloud
└── ESO ExternalSecrets              ← infra-cloud

rhbk-operator namespace (Keycloak — deployed by infra-cloud)
├── Keycloak (RHBK operator CRD, TLS via cert-manager)
└── KeycloakRealmImport (nico realm, nico-rest + ncx-service clients)

nico-system namespace (per-site, edge — upstream nico + site-agent charts)
├── nico-api (Core gRPC API)         ← upstream
├── nico-dhcp, nico-dns, nico-pxe    ← upstream
├── nico-ssh-console-rs              ← upstream
├── nico-hardware-health             ← upstream
├── nico-dsx-exchange-consumer       ← upstream
├── nico-bmc-proxy                   ← upstream
├── nico-flow (PSM/NSM sidecars)     ← upstream
├── nico-rest-site-agent             ← upstream (connects to cloud Temporal)
├── NATS (MQTT for DSX Exchange)     ← infra-site
├── Vault HA (3-node Raft)           ← infra-site
├── nico-site-pg (Crunchy PG15)      ← infra-site (nico, flow, psm, nsm DBs)
└── ESO ExternalSecrets              ← infra-site
```

### TLS Architecture

Self-signed CA chain via cert-manager, with Vault PKI for Core services:

```
nico-bootstrap-issuer (self-signed)
  └── nico-root-ca Certificate (10-year CA, ECDSA P-256)
        ├── nico-rest-ca-issuer (ClusterIssuer) → REST services, Temporal certs
        ├── site-issuer (ClusterIssuer) → Vault TLS certs (pre-Vault bootstrap)
        └── imported into Vault PKI (nicoca mount)
              └── vault-nico-issuer (Vault PKI ClusterIssuer) → Core services
```

### Vault Configuration

- **Primary:** 3-node HA Raft cluster with TLS, cert-reload sidecar (UBI-minimal)
- **Unseal:** `make vault-init` (one-time), postStart auto-unseal from K8s Secret
- **PKI:** `nicoca` engine with `nico-cluster` role (SPIFFE URI SANs)
- **Auth:** AppRole for Core API (`nico-vault-policy`), K8s auth for cert-manager
- **KV:** `secrets/` mount with factory-default BMC credential seeds
- **Flow tokens:** Periodic tokens for PSM/NSM with scoped policies
- **Production unseal:** Replace postStart with cloud KMS (AWS/GCP/Azure) or
  external Transit Vault (in NICo Cloud or operated by security team)

### Prerequisites (installed by prereqs chart)

| Component | OLM source | Purpose |
|---|---|---|
| cert-manager | `redhat-operators` | TLS certificate lifecycle |
| Crunchy PostgreSQL | `certified-operators` | Managed PostgreSQL clusters |
| RHBK (Keycloak) | `redhat-operators` | Identity and access management |
| External Secrets Operator | `redhat-operators` | Cross-namespace secret sync |

## Directory Structure

```
docker/ubi/                          UBI-based Dockerfiles for all NICo images
helm/
  vendor/infra-controller/           Upstream git submodule (read-only)
  nvidia-infra-controller-prereqs/   OLM operators + ClusterIssuers + ESO
  values/                            Values overrides for upstream charts
    nico-rest.yaml                     REST API, workflow, site-manager, credsmgr
    nico-core.yaml                     Core tier (all services enabled)
    nico-rest-site-agent.yaml          Site-agent (Temporal client)
  infra-cloud/                       Red Hat cloud infrastructure add-ons
    Chart.yaml                         Depends on: Temporal chart
    templates/
      cnpg-cluster.yaml                Consolidated PG (nico, temporal, keycloak)
      keycloak/                        RHBK Keycloak CRDs + TLS
      routes.yaml                      OpenShift Routes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rh-ecosystem-edge/ncp-dsx-nico-deploy](https://github.com/rh-ecosystem-edge/ncp-dsx-nico-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
