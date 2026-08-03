---
trigger: always_on
description: KubeVPN is a CLI tool that connects local development environments to Kubernetes cluster networks. It creates TUN
---

# CLAUDE.md — KubeVPN Project Instructions

## Project Overview

KubeVPN is a CLI tool that connects local development environments to Kubernetes cluster networks. It creates TUN
devices, manages DHCP IP allocation, injects Envoy sidecar proxies, and supports SSH jump hosts.

- **Language:** Go 1.26+
- **Module:** `github.com/wencaiwulue/kubevpn/v2`
- **Entry point:** `cmd/kubevpn/main.go`

## Build & Test

```bash
# Build
go build ./...              # verify compilation
make kubevpn                # build binary with ldflags

# Test
go test ./pkg/...           # run no-cluster tests (no kubeconfig needed)
go test -tags=integration ./pkg/...  # also run tests that need a real cluster
go test ./pkg/inject/... -v # test specific package
go vet ./pkg/...            # static analysis

make ut             # FULL suite incl. cluster tests (-tags=integration); needs kubeconfig
make ut-no-cluster  # ONLY no-kubeconfig tests; used by the Windows CI job

# Note: TestPing always fails in this environment (needs raw socket)
# Note: CGO_ENABLED=0, so -race is unavailable

# Generate protobuf
make gen
```

### Test Build Tags (kubeconfig vs no-kubeconfig)

Tests that need a real Kubernetes cluster (and therefore a kubeconfig) live behind
the **`integration`** build tag — `//go:build integration` at the top of the file.
Everything without that tag must run with NO kubeconfig (use `fake.NewSimpleClientset()`,
`httptest`, temp/dummy kubeconfig files, or `net.Pipe()`). The Windows CI job has no
cluster and runs `make ut-no-cluster`, so:

- **Never** make a non-`integration` test call a real cluster API (`ToRESTConfig()` +
  real Get/List, `kubernetes.NewForConfig` against a live server). If it needs a cluster,
  move it to a `//go:build integration` file (e.g. `*_integration_test.go`).
- A non-`integration` test that builds a `genericclioptions.ConfigFlags` MUST point
  `configFlags.KubeConfig` at a temp/dummy/nonexistent path — never the ambient default.
- Other tag in use: `tun` for real-TUN tests needing `CAP_NET_ADMIN` (run on demand).

## Test Cluster

A test cluster is available at `/data/.kube/config`:

- Context: `kubernetes-admin-cf2096c3e67fb41a0b0cfc3de9a72f027` (Alibaba Cloud ACK)
- The default context `orbstack` is local and unreachable — always switch to the remote context first

## Architecture

```
cmd/kubevpn/cmds/     CLI command definitions (DO NOT MODIFY in refactoring)
pkg/
├── config/            Constants, image config, syncthing paths
├── controlplane/      Envoy xDS control plane (gRPC, config cache)
├── core/              Network protocol core (TUN, gvisor, TCP/UDP forwarding)
├── cp/                File copy (kubectl cp equivalent)
├── daemon/            gRPC daemon server
│   ├── action/        Per-command daemon handlers (connect, proxy, leave, etc.)
│   │   ├── connection.go  Connection lookup/remove helpers (findConnection, removeConnection, cleanupConnection)
│   │   ├── lifecycle.go   SessionLifecycle — session context + Teardown (cancel + logger detach) for daemon sessions
│   │   └── writer.go      newStreamWriter, initStreamLogger, resolveKubeconfigBytes
│   ├── handler/       WebSocket SSH terminal handler
│   ├── elevate/       Privilege escalation (sudo/admin)
│   └── rpc/           Generated protobuf (DO NOT EDIT *.pb.go)
├── dhcp/              DHCP IP allocation via ConfigMap
├── dns/               DNS setup (platform-specific: linux/unix/windows)
├── driver/            TUN/TAP driver management (wintun, openvpn)
├── handler/           Core business logic
│   ├── session_base.go    SessionBase — shared base (K8sClient + rollbackList + cleanup) embedded by ConnectOptions & DataSession
│   ├── connect.go         ConnectOptions (= ControlSession) — control-plane methods + data-plane stubs
│   ├── control_session.go type ControlSession = ConnectOptions (alias)
│   ├── data_session.go    DataSession — data-plane methods, DoConnect, cleanupDataPlane
│   ├── connection.go      Connection interface + compile-time assertions (both types satisfy it)
│   ├── rollback.go        rollbackList — mutex-guarded rollback registry (embedded by SessionBase & SyncOptions)
│   ├── cleaner.go         ConnectOptions.Cleanup + cleanupControlPlane + executeRollbackFuncs
│   ├── connect_tun.go     TUN server, port forwarding, health checks
│   ├── connect_route.go   Dynamic routing, extra routes, watchAndRoute
│   ├── connect_dns.go     DNS setup
│   ├── connect_upgrade.go Traffic manager deployment upgrade
│   ├── network.go         NetworkManager — owns full networking lifecycle (port-forward, TUN IP allocation, routes, DNS)
│   ├── k8s_client.go      K8sClient embedded struct (via SessionBase → ConnectOptions/DataSession/SyncOptions)
│   ├── traffmgr.go        Create traffic manager pod
│   ├── traffmgr_resources.go  K8s resource generators (deploy, svc, secret, etc.)
│   ├── leave.go           Leave/unpatch proxy resources
│   ├── proxy.go           Port mapping management
│   ├── proxy_mapper.go    Mapper for port-forward config from ConfigMap
│   ├── sync.go            Syncthing-based file sync
│   └── reset.go           Reset workloads to original spec
├── inject/            Sidecar injection (Injector interface + Strategy pattern)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubenetworks/kubevpn](https://github.com/kubenetworks/kubevpn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
