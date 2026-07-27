---
trigger: always_on
description: The cluster-etcd-operator (CEO) manages the etcd cluster in OpenShift, handling:
---

# AI agent instructions for the cluster-etcd-operator codebase.

## What This Repo Is

The cluster-etcd-operator (CEO) manages the etcd cluster in OpenShift, handling:
- etcd cluster scaling during bootstrap and steady state operation
- TLS certificate provisioning and rotation
- Backup and disaster recovery
- Static pod lifecycle management via library-go framework
- etcd health monitoring and defragmentation
- Bootstrap teardown and scaling strategies (HA, Delayed-HA, Two-Node, etc.)

See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed architecture documentation.

## Critical Rules

1. **Never edit `vendor/` files** - managed by `go mod vendor`
2. **Never edit generated files** - anything in `vendor/` or matching `zz_generated.*` patterns
3. **Always run `make verify` before committing** - includes linting, generated file checks, and vendor verification
4. **Always run `go mod tidy && go mod vendor`** after dependency changes
5. **Never modify manifests without regenerating** - alerts/dashboards are generated from jsonnet (see [Alerts and Dashboards](#alerts-and-dashboards))
6. **Always use `make build`** not `go build` - for proper version injection

## Repository Structure

```
cluster-etcd-operator/
├── bindata/             # Embedded YAML manifests and scripts
│   ├── bootkube/        # Bootstrap manifests
│   ├── etcd/            # etcd manifests and scripts (cluster backup, cluster restore, quorum restore)
│   └── tnfdeployment/   # Two Node Fencing (TNF) manifests
├── cmd/
│   ├── cluster-etcd-operator/            # Main operator binary
│   ├── cluster-etcd-operator-tests-ext/  # OTE test binary
│   ├── tnf-monitor/                      # Two Node Fencing monitor
│   └── tnf-setup-runner/                 # Two Node Fencing setup
├── docs/                # Documentation (HACKING, FAQ, overview)
├── hack/                # Build and generation scripts
├── jsonnet/             # Alert and dashboard definitions
├── manifests/           # CVO-managed YAML (deployment, RBAC, monitoring)
├── pkg/
│   ├── operator/        # Core operator logic (20+ controllers)
│   ├── etcdcli/         # etcd client wrapper
│   ├── tlshelpers/      # TLS certificate utilities
│   └── dnshelpers/      # DNS resolution helpers
└── test/e2e/            # End-to-end tests
```

## Key Patterns to Follow

### Controller Pattern

Controllers follow the library-go factory pattern:

```go
func NewMyController(
    livenessChecker *health.MultiAlivenessChecker,
    operatorClient v1helpers.StaticPodOperatorClient,
    eventRecorder events.Recorder,
) factory.Controller {
    c := &MyController{
        operatorClient: operatorClient,
    }

    syncer := health.NewDefaultCheckingSyncWrapper(c.sync)
    livenessChecker.Add("MyController", syncer)

    return factory.New().
        ResyncEvery(time.Minute).
        WithInformers(
            operatorClient.Informer(),
        ).
        WithSync(syncer.Sync).
        ToController("MyController", eventRecorder.WithComponentSuffix("my-controller"))
}
```

Key elements:
- Constructor function named `NewXxxController`
- Wraps sync function with health checker
- Uses library-go factory builder
- Returns `factory.Controller` interface
- **All resources the controller reads must have a corresponding informer registered via `.WithInformers()`** — missing informers are one of the most common bugs in this codebase. When adding or modifying a controller, verify that every lister/getter used in the sync function has its informer wired up in the factory builder, otherwise the controller will operate on stale data

### Status Conditions

Use `v1helpers.UpdateStatus` to set conditions:

```go
_, _, updateErr := v1helpers.UpdateStatus(ctx, c.operatorClient,
    v1helpers.UpdateConditionFn(operatorv1.OperatorCondition{
        Type:    "MyControllerDegraded",
        Status:  operatorv1.ConditionTrue,
        Reason:  "SyncError",
        Message: err.Error(),
    }),
)
```

Guidelines for status conditions:
- **Use `Degraded` very sparingly** — only set it after a meaningful amount of time has passed (etcd moves slowly due to static pod machinery) and only when it requires **actionable** human intervention
- **Avoid conditions that flap** — always add an inertia component (e.g., require the error state to persist for several sync cycles before reporting Degraded). The operator has historically bumped inertia from 5 to 10 minutes to reduce flapping during rollouts.

### Bootstrap Scaling Strategies

Openshift supports anywhere from 1 to 5 control-plane nodes depending on the level of high availability needed. The bootstrap scaling strategy describes the invariants which will be enforced when scaling the etcd cluster.

The operator enforces different quorum requirements during bootstrap:
- `HAScalingStrategy` - requires at least 3 nodes to scale up (default)
- `DelayedHAScalingStrategy` - allows 2 nodes during bootstrap, then requires 3
- `TwoNodeScalingStrategy` - requires 2 nodes (for Two Node OpenShift with Fencing deployments)
- `DelayedTwoNodeScalingStrategy` - allows 1 node during bootstrap, then requires 2 (for Two Node OpenShift with Fencing deployments)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/cluster-etcd-operator](https://github.com/openshift/cluster-etcd-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
