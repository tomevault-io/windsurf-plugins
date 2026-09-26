---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code and others) when working with code in this repository. `AGENTS.md` is an exact copy of this file; `make check-agents-sync` keeps the two in sync.
---

# CLAUDE.md

This file provides guidance to AI coding agents (Claude Code and others) when working with code in this repository. `AGENTS.md` is an exact copy of this file; `make check-agents-sync` keeps the two in sync.

## Build & Test Commands

```bash
make manifests generate   # MUST run after editing *_types.go or kubebuilder markers
make test                 # Unit tests + integration tests (requires envtest)
make test-integration     # Integration tests only (envtest + golden files)
make lint                 # golangci-lint
make lint-fix             # Auto-fix lint issues
make build                # Build binary to bin/manager
make docker-build         # Build Docker image
make helm-package         # Package the Helm chart
```

Run a single unit test:
```bash
go test ./pkg/workload/ -run TestAdapterForSpec -v
```

Run a single integration test case:
```bash
KUBEBUILDER_ASSETS="$(bin/setup-envtest use -p path)" \
  go test ./cmd/integration/ -v -timeout 300s -count=1 -run TestIntegration/reconcile/job-checkpoint-restart
```

Update golden files after intentional changes. Unit and integration golden files regenerate separately:
```bash
TESTUTIL_UPDATE_EXPECTED=true go test ./pkg/report/   # unit goldens, per package
TESTUTIL_UPDATE_EXPECTED=true make test-integration   # cmd/integration/testdata/ only
```

### UAT Tests (Kind + KWOK)

UAT tests validate rendered catalog output against golden files using a Kind cluster with KWOK-simulated GPU nodes. Requires: Docker, Kind, Tilt, Go 1.24+.

```bash
# Full lifecycle (CI):
make setup-test-uat     # Create Kind cluster
make tilt-uat-ci        # Deploy infra (Kubeflow Trainer, KWOK, controller) — headless
make test-uat-run       # Run UAT tests against deployed cluster
make cleanup-test-uat   # Delete Kind cluster

# One-shot (handles everything):
make test-uat
```

For development iteration (faster feedback loop):
```bash
make setup-test-uat              # Once
make tilt-uat                    # Interactive Tilt (keep in another terminal, hot-reloads)
make test-uat-run                # Run as needed
```

Run a single UAT test:
```bash
kind get kubeconfig --name nvcre-test-uat > /tmp/kind-uat.kubeconfig
KUBECONFIG=/tmp/kind-uat.kubeconfig NVCRECTL=bin/nvcrectl \
  go test -tags=uat ./test/uat/ -v -timeout 900s -count=1 -run TestAWSGB200NCCL
```

UAT golden files are in `test/uat/testdata/<csp>/<gpu>/nccl/expected_pods.yaml`. When catalog entries change, tests write `.actual` files on failure — review the diff, then copy `.actual` over the golden file if correct.

**Note**: If `tilt-uat-ci` fails with Kubeflow Trainer timeout, retry — it's a race condition on first deploy. The second run typically succeeds since Kubeflow is already running.

## Architecture

Kubebuilder-based Kubernetes controller for GPU cluster burn-in certification. Single binary, six reconcilers.

### CRD Hierarchy (Certification → Workflow → Job)

Follows the Deployment → ReplicaSet → Pod composition pattern:

- **Certification** creates one Workflow per category from the catalog. Records failed nodes per category with a reason. Workflow named `<certName>-<domain>-<variant>`.
- **Workflow** creates a single Job from `spec.jobTemplate`. Applies orchestration target, manages iterations, creates dependency resources. Job named `<workflowName>-job`.
- **Job** creates a workload (TrainJob) via the adapter pattern. Manages health monitoring, goodput measurement, checkpoint restart.
- **GoodputMeasurement** watches a Job, parses pod logs via LogProfile regex patterns, computes goodput ratio.
- **BandwidthMeasurement** watches a Job, parses NCCL log output, and computes per-bus bandwidth metrics.
- **LogProfile** (cluster-scoped) defines regex patterns with named capture groups for parsing training framework logs.

There is no Remediation controller. ADR-061 removed it. NVCRE does not taint, cordon, or patch nodes; it records failed nodes with a reason (`HardwareFailureDetected`, `ThresholdViolation`, or `WorkloadFailed`) in the Certification status.

### Key Packages

- `pkg/workload/` — Adapter interface normalizing five training frameworks to `WorkloadPhase` (Running/Succeeded/Failed). `ForSpec()` factory selects adapter based on which `WorkloadSpec` field is set.
- `pkg/catalog/` — Maps `{domain, variant}` → WorkflowSpec builder via `init()` registration. Adding a category = one new Go file.
- `pkg/nodemonitor/` — `NodeFailureDetector` interface; the CEL implementation evaluates expressions against Node objects. Detectors are selected by which typed field is set on `Job.spec.nodeHealthMonitor` (same pattern as workload adapters), not by name lookup.
- `pkg/goodput/` — Log parsing (`parser.go`), goodput calculation (`calculator.go`), pod log reading (`reader.go` with `PodLogFetcher` interface for test injection).
- `pkg/orchestration/` — Node partitioning: simple (name-sorted chunking), topology-aware (greedy domain-based allocation), and bisection (used internally by `diagnose` for adaptive fault isolation).
- `pkg/gpu/` — GPU architecture detection and defaults (gpusPerNode, MNNVL).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/cluster-readiness-engine](https://github.com/NVIDIA/cluster-readiness-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
