---
trigger: always_on
description: JaisCloud is a local cloud emulator written in Go. Each cloud ships as its own self-contained binary (`jaiscloud-aws`, `jaiscloud-azure`, `jaiscloud-gcp`). The binary IS the cloud — there is no runtime `--cloud` flag and no shared provider logic between clouds. AWS wire protocols (Query/XML, JSON/Target, REST) are implemented so any AWS SDK can point at `jaiscloud-aws` without modification.
---

# JaisCloud — Developer Reference

## Project

JaisCloud is a local cloud emulator written in Go. Each cloud ships as its own self-contained binary (`jaiscloud-aws`, `jaiscloud-azure`, `jaiscloud-gcp`). The binary IS the cloud — there is no runtime `--cloud` flag and no shared provider logic between clouds. AWS wire protocols (Query/XML, JSON/Target, REST) are implemented so any AWS SDK can point at `jaiscloud-aws` without modification.

**Design decision: per-cloud binaries, not a cloud-agnostic core.**
Every cloud has its own adapter, its own provider implementations, and its own entry point. Only infrastructure utilities are shared (`gateway`, `store`, `model`, `admin`, `blobfs`, `clock`, `events`, `executor`, `config`). When Azure or GCP services are implemented, they will live entirely in `internal/azure/` or `internal/gcp/` — no provider code will be shared with AWS.

**Implemented AWS services:** SQS, SNS, IAM/STS, DynamoDB (+ Streams), S3, Lambda, KMS, SecretsManager, SSM, API Gateway, CloudFormation, EMR (on EC2), EMR on EKS, EventBridge, CloudWatch, EKS, EC2, Route53, RDS, ElastiCache, ECS, Glue.

---

## Module

```
module jaiscloud   # go.mod
go 1.26.3
```

---

## Directory layout

```
cmd/
  jaiscloud-aws/main.go    # AWS binary — full provider wiring, Cobra CLI
  jaiscloud-azure/main.go  # Azure binary stub (501 for all ops)
  jaiscloud-gcp/main.go    # GCP binary stub (501 for all ops)
docs/                      # Architecture, LLD, design documents
internal/
  # ── SHARED INFRASTRUCTURE (cloud-neutral utilities) ──────────────────────
  adapter/
    adapter.go             # CloudAdapter + Codec interfaces only — no implementations
  provider/
    provider.go            # HandlerFunc type, OK() helper
    registry.go            # Registry — Dispatch (exact match → error)
  model/                   # NormalizedRequest, ProviderResponse, ProviderError, Cloud enum
  gateway/                 # HTTP server (Chi), middleware, request dispatch
    server.go              # Server — holds single CloudAdapter; handleCloudRequest
    middleware/            # Recovery, RequestID, Logging, Metrics
  admin/                   # /_jaiscloud/* endpoints; Resetter, Snapshotter interfaces
  blobfs/                  # BlobStore (Memory/LocalFS); BlobFetcher (S3BlobFetcher)
  clock/                   # Clock interface: RealClock, FixedClock, OffsetClock
  config/                  # Config struct; Viper loading; env prefix JAISCLOUD_
  events/                  # In-process EventBus (subscribe/publish)
  executor/                # Container executor abstraction
    lambda/                # Lambda executor (mock / docker / k8s)
  k8shelpers/              # Generic K8s helpers (BuildPodSpec, IdentityMutator, OwnershipPatcher)
  k8stypes/                # K8s type defs
  sparkhelpers/            # Spark-specific K8s helpers
  platform/                # PlatformConfig — TLS init containers, env fragments, volume mounts
  reqctx/                  # Request context helpers
  resourcemgr/             # Deletion guards: CheckParent, AcquireDelete, DeleteGuardRule
  certstore/               # TLS cert storage
  snapshottypes/           # Snapshotter/Resetter/PostRestoreHook interfaces (cycle-breaking)
  persistence/
    version/               # CodeSnapshotVersion, CodeDBSchemaVersion, Envelope struct, FingerprintKEK
    snapshot/              # SnapshotLoop — periodic state.json persistence
    platform/              # flock_darwin.go / flock_linux.go — data-dir OS lock
  store/                   # ResourceStore interface + memory/postgres implementations
    migrations/            # SQL migration files (001–015)
    object/                # Generic ObjectStore
    stream/                # MemoryStreamStore (DynamoDB Streams)
  workers/                 # Worker interface + Registry — centralised lifecycle for background goroutines

  # ── AWS-SPECIFIC (internal/aws/) ─────────────────────────────────────────
  aws/
    adapter/               # AWSAdapter, router.go, services.go
      services/            # Per-service Codec implementations (27 files)
    provider/              # All AWS business logic
      apigw/               # APIGatewayProvider
      cache/               # ElastiCache (metadata only)
      catalog/             # Glue Data Catalog provider
      cloudwatch/          # CloudWatchProvider — metrics ring, alarms
        logs/              # CloudWatch Logs
      compute/             # EC2 (metadata only)
      container/           # ECS (metadata only)
      dns/                 # Route53 (metadata only)
      eks/                 # EKS (metadata only)
      emr/                 # EMRProvider — RunJobFlow, steps, bootstrap, Spark K8s/Docker
      emroneks/            # EMRContainersProvider — virtual clusters, job runs
      events/              # EventBridgeProvider
      function/            # FunctionProvider — Lambda
      iam/                 # IAMProvider + STS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaisrajms/jaiscloud](https://github.com/jaisrajms/jaiscloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
