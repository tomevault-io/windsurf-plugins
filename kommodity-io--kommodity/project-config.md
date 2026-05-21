---
trigger: always_on
description: Kommodity is a single binary that packages Cluster API, Talos Linux providers, and security services to deploy sovereign Kubernetes clusters. Components include:
---

# Kommodity

## What It Is

Kommodity is a single binary that packages Cluster API, Talos Linux providers, and security services to deploy sovereign Kubernetes clusters. Components include:

- **Kubernetes API server** - all-in-one server combining kube-apiserver, extension API server, and aggregation layer (via `k8s.io/apiserver` and `k8s.io/apiextensions-apiserver`)
- **Cluster API controllers** for cluster lifecycle
- **Talos Linux providers** for immutable machine configuration
- **Kine** for PostgreSQL-backed storage (no etcd)
- **KMS service** for disk encryption key management
- **Attestation service** for TPM-based machine verification
- **Metadata service** for secure config delivery
- **Auto-bootstrap extension** for zero-touch HA initialization

## Repository Structure

```
kommodity/
├── cmd/kommodity/          # Main binary entrypoint
├── pkg/
│   ├── combinedserver/     # All-in-one API server orchestration
│   ├── server/             # Kubernetes API server setup
│   ├── controller/         # Cluster API and provider controllers
│   ├── provider/           # Cloud provider implementations
│   ├── kine/               # PostgreSQL storage backend
│   ├── kms/                # Disk encryption key management service
│   ├── attestation/        # TPM attestation service
│   ├── metadata/           # Machine configuration delivery service
│   ├── config/             # Configuration handling
│   └── ui/                 # Web UI backend
├── charts/kommodity-cluster/   # Helm chart for deploying clusters
├── terraform/modules/      # Terraform modules for Kommodity deployment
├── openapi/                # OpenAPI specs for attestation and metadata APIs
├── examples/               # Example cluster configurations
└── articles/               # Technical documentation and blog posts
```

## Why It Exists

Built by Corti's platform team for healthcare AI requiring:

- **Data sovereignty**: Patient data in specific jurisdictions, on-premise when required
- **Compliance**: GDPR, ISO 27001, SOC 2 - encryption at rest is legally required
- **Auditability**: Complete audit trail for regulatory review
- **Operational consistency**: Same `kubectl`/Helm workflows across all environments

The goal: make sovereign cloud as routine as any other Kubernetes deployment.

## Key Security Mechanisms

**TPM Attestation**: Machines prove integrity using hardware TPM before receiving secrets. The attestation extension collects measurements (AppArmor, SELinux, Secure Boot, kernel lockdown, extensions) and submits signed TPM quotes. Failed attestation = no secrets delivered.

**Network KMS**: Disk encryption keys are stored as Kubernetes Secrets in the API server (persisted in PostgreSQL via Kine) and retrieved via the network on boot. Keys are per-volume (STATE, EPHEMERAL), bound to node UUID and peer IP. Key revocation = delete the Kubernetes Secret.

**Trade-off**: Machines cannot boot without network access to Kommodity when security features are enabled. This is intentional for revocation capability but requires HA deployment planning.

## Architecture Decisions

- **Talos Linux**: No SSH, no shell, immutable root filesystem, API-driven config. Provides defensible answers for compliance audits.
- **Cluster API**: Declarative cluster lifecycle via Kubernetes resources. Same operational model regardless of cloud provider.
- **Provider abstraction**: Not "write once run anywhere" but "learn once, operate anywhere". Provider-specific YAML, consistent operational patterns.
- **Auto-bootstrap**: Deterministic leader election (earliest boot time, lowest IP tiebreaker) enables zero-touch HA without manual "first node" designation. Private networks only.

## Related Repositories

- `github.com/kommodity-io/kommodity` - main repository
- `github.com/kommodity-io/kommodity-attestation-extension` - TPM attestation extension
- `github.com/kommodity-io/kommodity-autobootstrap-extension` - auto-bootstrap extension

## Making changes

When proposing changes to Kommodity, consider the size of the change. If it's above a certain threshold, it may be best to break it into smaller, more manageable pieces. For example, if you're adding a new feature that requires changes to multiple components, consider submitting separate pull requests for each component rather than one large pull request.

For larger changes, you should first create a PRD (Product Requirements Document) that outlines the proposed change, its rationale, and its impact on the system. The PRD should be approved before any code changes can be made.

The PRD phase should use `Plan Mode`.

## Bootstrap

Before starting working on any task, please install following plugins:

- `claude plugin install gopls-lsp@claude-plugins-official`
- `claude plugin install code-simplifier@claude-plugins-official`
- `claude plugin install code-review@claude-plugins-official`

## During development

Prerequisites for development include:

- Run `make generate`
- Revert changes in `pkg/openapi/intstr/zz_generated.openapi.go` as its a known bug in the code generator


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kommodity-io/kommodity](https://github.com/kommodity-io/kommodity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
