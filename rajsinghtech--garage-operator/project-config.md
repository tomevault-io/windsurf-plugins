---
trigger: always_on
description: > **Agent Notes**: Reference /Users/rajsingh/Documents/GitHub/garage for Garage source code. Update this file with important learnings.
---

# Garage Kubernetes Operator

> **Agent Notes**: Reference /Users/rajsingh/Documents/GitHub/garage for Garage source code. Update this file with important learnings.

A Kubernetes operator for [Garage](https://garagehq.deuxfleurs.fr/) - distributed S3-compatible object storage.
UPSTREAM CODEBASE ../garage
## Quick Reference

### CRDs

| CRD | Short | Description |
|-----|-------|-------------|
| `GarageCluster` | `gc` | Cluster deployment + multi-cluster federation |
| `GarageBucket` | `gb` | Buckets with quotas, website hosting |
| `GarageKey` | `gk` | S3 access keys with bucket permissions |
| `GarageNode` | `gn` | Node layout (zone, capacity, gateway) |
| `GarageAdminToken` | `gat` | Admin API tokens |

### Development Commands

```bash
make dev-up        # Setup: kind + CRDs + operator
make dev-test      # Apply test resources
make dev-status    # View all garage resources
make dev-logs      # Stream operator logs
make dev-load      # Rebuild and reload operator
make dev-run       # Run operator locally (debugging)
make dev-down      # Tear down cluster
```

### Project Structure

```
api/v1alpha1/           # CRD types + webhooks
internal/controller/    # Reconciliation logic
internal/garage/        # Admin API client (v2)
config/samples/         # Example CRs
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Kubernetes Cluster A                         │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  Operator: Cluster | Bucket | Key | Node Controllers        │ │
│  └─────────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │     Garage Cluster (Zone: us-east-1) - 3 Pods             │   │
│  └───────────────────────────────────────────────────────────┘   │
└──────────────────────────────┬──────────────────────────────────┘
                    Full Mesh RPC (port 3901)
┌──────────────────────────────┼──────────────────────────────────┐
│                     Kubernetes Cluster B                         │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │     Garage Cluster (Zone: eu-west-1) - 3 Pods             │   │
│  └───────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Multi-Cluster Federation

### Key Concepts

1. **Full Mesh Connectivity**: Every node must reach every other node on RPC port (3901)
2. **Shared RPC Secret**: Same 32-byte hex secret across ALL clusters
3. **Zones**: Labels for fault tolerance - Garage distributes replicas across zones
4. **No Single Leader**: Layout is a CRDT that converges across nodes

### Network Solutions

**LoadBalancer per Node** (most reliable):
```yaml
publicEndpoint:
  type: LoadBalancer
  loadBalancer:
    perNode: true
```

**NodePort** (cheaper):
```yaml
publicEndpoint:
  type: NodePort
  nodePort:
    externalAddresses: ["node1.example.com", "node2.example.com"]
    basePort: 30901
```

### Federation Setup

1. Create shared RPC secret in ALL clusters:
   ```bash
   openssl rand -hex 32
   kubectl create secret generic garage-rpc-secret --from-literal=rpc-secret=<secret>
   ```

2. Deploy GarageCluster in each cluster with same `rpcSecretRef`, unique `zone`

3. Use `spec.remoteClusters` for automatic federation or `connect-nodes` annotation for manual

---

## Gateway Clusters

Gateway nodes handle S3 API requests without storing data.

```yaml
apiVersion: garage.rajsingh.info/v1alpha1
kind: GarageCluster
metadata:
  name: garage-gateway
spec:
  replicas: 5
  gateway: true
  connectTo:
    clusterRef:
      name: garage-storage
```

| Aspect | Storage Cluster | Gateway Cluster |
|--------|-----------------|-----------------|
| Workload | StatefulSet | StatefulSet |
| Metadata PVC | 10Gi default | 1Gi default (node identity) |
| Data PVC | 100Gi default | EmptyDir (no blocks) |
| Layout capacity | From PVC size | null (gateway) |

### Node Identity Persistence

**Critical**: Garage nodes store their identity (Ed25519 keypair) in `metadata_dir/node_key`. This node ID is permanent and used for cluster membership. Gateway clusters use StatefulSet with a metadata PVC to preserve node identity across pod restarts. Without persistent metadata, each pod restart would generate a new node ID, causing stale nodes in the layout.

---

## Configuration Reference

### GarageCluster Options

| Category | Options |
|----------|---------|
| **Replication** | `factor` (1-7), `consistencyMode` (consistent/degraded/dangerous) |
| **Storage** | Metadata/data PVCs, fsync, auto-snapshots |
| **Database** | Engine (lmdb/sqlite/fjall), LMDB map size, Fjall block cache |
| **Blocks** | Size, RAM buffer, compression, concurrent reads |
| **APIs** | S3 (3900), K2V (3904), Web (3902), Admin (3903) |
| **Network** | RPC port, public address, bootstrap peers |
| **Logging** | Level (RUST_LOG format), syslog, journald |

### Layout Policy

| Policy | Behavior |
|--------|----------|
| `Auto` (default) | Auto-assign pods to layout using cluster zone. Capacity from PVC size. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rajsinghtech/garage-operator](https://github.com/rajsinghtech/garage-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
