---
trigger: always_on
description: **Talos Pilot** is a terminal UI (TUI) for managing and monitoring Talos Linux Kubernetes clusters. It provides real-time diagnostics, log streaming, network analysis, and cluster health monitoring.
---

# CLAUDE.md - Talos Pilot Development Guide

## Project Overview

**Talos Pilot** is a terminal UI (TUI) for managing and monitoring Talos Linux Kubernetes clusters. It provides real-time diagnostics, log streaming, network analysis, and cluster health monitoring.

### Crate Structure

```
crates/
├── talos-rs/           # Low-level Talos gRPC client library
├── talos-pilot-core/   # Shared business logic (~1,760 lines, 47 tests)
└── talos-pilot-tui/    # Terminal UI application (ratatui-based)
```

### Key Technologies
- **Rust 2024 edition**
- **Async runtime:** tokio
- **TUI framework:** ratatui + crossterm + tachyonfx
- **gRPC client:** tonic + prost
- **Kubernetes client:** kube-rs
- **Error handling:** color-eyre + thiserror

---

## Feature Status

### Implemented Features

| Feature | Component | Status |
|---------|-----------|--------|
| Multi-cluster overview | `cluster.rs` | Complete |
| Node details (CPU, memory, load) | `cluster.rs` | Complete |
| Service logs with search | `logs.rs` | Complete |
| Multi-service logs (Stern-style) | `multi_logs.rs` | Complete |
| Process tree view | `processes.rs` | Complete |
| Network stats, KubeSpan, packet capture | `network.rs` | Complete |
| etcd status & quorum | `etcd.rs` | Complete |
| K8s workload health | `workloads.rs` | Complete |
| System diagnostics | `diagnostics/` | Complete |
| CNI detection (Flannel/Cilium/Calico) | `diagnostics/cni/` | Complete |
| Addon detection | `diagnostics/addons/` | Complete |
| Security/PKI audit | `security.rs` | Complete |
| Lifecycle (versions, config drift) | `lifecycle.rs` | Complete |
| Node operations (drain/reboot) | `node_operations.rs` | Complete |
| Rolling operations | `rolling_operations.rs` | Complete |
| Audit logging | `audit.rs` | Complete |

### Planned Features

| Feature | Priority | Notes |
|---------|----------|-------|
| Container namespace support | Medium | Show pod/container names for connections |
| Upgrade availability alerts | Low | Check for new Talos/K8s versions |

---

## Core Modules (talos-pilot-core)

| Module | Lines | Tests | Purpose |
|--------|-------|-------|---------|
| `indicators` | ~300 | 6 | HealthIndicator, HasHealth trait, QuorumState, SafetyStatus |
| `formatting` | ~280 | 10 | format_bytes, format_duration, format_percent, pluralize |
| `selection` | ~320 | 6 | SelectableList<T>, MultiSelectList<T> for UI navigation |
| `async_state` | ~200 | 7 | AsyncState<T> for loading/error/refresh management |
| `errors` | ~180 | 4 | format_talos_error, ErrorCategory, user-friendly messages |
| `network` | ~180 | 6 | port_to_service, connection classification |
| `diagnostics` | ~200 | 5 | CheckStatus, CniType, CniInfo, PodHealthInfo |
| `constants` | ~100 | 3 | Thresholds, CRD lists, refresh intervals |

---

## Talos Linux Reference

### Network Ports

| Port | Protocol | Service | Used By |
|------|----------|---------|---------|
| 50000 | TCP | apid (Talos API) | talosctl, control plane nodes |
| 50001 | TCP | trustd | Worker nodes for TLS certs |
| 6443 | TCP | kube-apiserver | kubectl, kubelets |
| 2379 | TCP | etcd client | kube-apiserver |
| 2380 | TCP | etcd peer | etcd cluster members |
| 10250 | TCP | kubelet | kube-apiserver |
| 10259 | TCP | kube-scheduler | Health checks |
| 10257 | TCP | kube-controller-manager | Health checks |

### Talos Machine API Methods

Key gRPC methods we use (all in `MachineService`):

| Method | Description |
|--------|-------------|
| `Version` | Get Talos version |
| `ServiceList` | List all services |
| `ServiceRestart` | Restart a service |
| `Logs` | Stream service logs |
| `Memory` | Memory usage |
| `LoadAvg` | CPU load averages |
| `CPUInfo` | CPU info |
| `Processes` | Process list |
| `NetworkDeviceStats` | Network interface stats |
| `Netstat` | Network connections |
| `Read` | Read file from node |
| `Dmesg` | Kernel ring buffer |
| `EtcdStatus` | etcd member status |
| `EtcdMemberList` | etcd cluster members |
| `EtcdAlarmList` | etcd alarms |
| `Kubeconfig` | Get kubeconfig |
| `ApplyConfiguration` | Apply config patch |
| `PacketCapture` | Capture packets (pcap) |
| `Reboot` | Reboot node |
| `Shutdown` | Shutdown node |

### COSI Resource API - NOT EXTERNALLY ACCESSIBLE

> **IMPORTANT:** The COSI State API is an **internal service** and is **NOT exposed** through port 50000.

**Do NOT attempt to implement COSI gRPC client code** - it will fail with `PermissionDenied`. Use `talosctl get` as a subprocess instead.

---

## Core Philosophies

### 1. State Over Logs

**The most important principle for diagnostics:**

> Check actual system state, not log messages. Logs are history; APIs and files are truth.

```rust
// GOOD: Direct state check
let healthy = client.read_file("/run/flannel/subnet.env").await.is_ok();

// GOOD: K8s API query for current state
let pods = kube_client.list::<Pod>(&params).await?;

// BAD: Log parsing for health determination
let logs = client.logs("kubelet", 100).await?;
let healthy = !logs.contains("error");  // DON'T DO THIS
```

### 2. Reliability Hierarchy

When implementing any check, prefer data sources in this order:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Handfish/talos-pilot](https://github.com/Handfish/talos-pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
