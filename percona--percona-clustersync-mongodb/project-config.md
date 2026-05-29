---
trigger: always_on
description: Percona ClusterSync for MongoDB (PCSM) replicates data between MongoDB clusters. Supports replica sets and sharded clusters with initial cloning and continuous change replication.
---

# AGENTS.md - Percona ClusterSync for MongoDB

Percona ClusterSync for MongoDB (PCSM) replicates data between MongoDB clusters. Supports replica sets and sharded clusters with initial cloning and continuous change replication.

## Prerequisites

### /etc/hosts

The MongoDB containers use hostnames that must resolve on the host. Add these entries to `/etc/hosts`:

```
# RS topology
127.0.0.1 rs00 rs01 rs02 rs10 rs11 rs12
# Sharded topology
127.0.0.1 src-rs00 src-rs10 src-rs20 src-cfg0 src-mongos
127.0.0.1 tgt-rs00 tgt-rs10 tgt-rs20 tgt-cfg0 tgt-mongos
```

### Tools

- Docker and Docker Compose
- Go (for building the binary)
- Python 3.13+ and [Poetry](https://python-poetry.org/) (for E2E tests and monitoring scripts)

Install Python dependencies:

```bash
poetry install
```

If `poetry run` fails with a "bad interpreter" error (e.g. after a Python version change), use the virtualenv directly: `.venv/bin/pytest` instead of `poetry run pytest`.

### Environment Variables

| Variable        | Default | Description                                            |
| --------------- | ------- | ------------------------------------------------------ |
| `MONGO_VERSION` | `8.0`   | MongoDB image version for test clusters                |
| `SRC_SHARDS`    | `2`     | Number of source shards (sharded topology only, max 3) |
| `TGT_SHARDS`    | `2`     | Number of target shards (sharded topology only, max 3) |

## Supported MongoDB Versions

| Source | Target | Topology    |
| ------ | ------ | ----------- |
| 6.0    | 6.0    | RS, Sharded |
| 7.0    | 7.0    | RS, Sharded |
| 8.0    | 8.0    | RS, Sharded |
| 6.0    | 7.0    | RS, Sharded |
| 6.0    | 8.0    | RS, Sharded |
| 7.0    | 8.0    | RS, Sharded |

Sharded entries (except 8.0 → 8.0) run a reduced E2E scope in CI; the full sharded suite is blocked by PCSM-255.
Downgrade (higher → lower) is not supported.

## Cluster Topology

### Connection URIs

| Topology | Source URI                   | Target URI                   |
| -------- | ---------------------------- | ---------------------------- |
| Sharded  | `mongodb://src-mongos:27017` | `mongodb://tgt-mongos:29017` |
| RS       | `mongodb://rs00:30000`       | `mongodb://rs10:30100`       |

**IMPORTANT**: Use these URIs exactly as shown. Do NOT append query parameters like `?replicaSet=rs0` or `?directConnection=true` — PCSM rejects `directConnection` and discovers topology automatically.

### Health Verification

After starting clusters, verify they are healthy:

**Sharded clusters:**

```bash
docker exec src-mongos mongosh --port 27017 --quiet --eval "db.adminCommand('ping')"
docker exec tgt-mongos mongosh --port 27017 --quiet --eval "db.adminCommand('ping')"
```

**RS clusters:**

```bash
docker exec rs00 mongosh --port 30000 --quiet --eval "rs.status().ok"
docker exec rs10 mongosh --port 30100 --quiet --eval "rs.status().ok"
```

## Commands

```bash
make build            # Production build
make test-build       # Debug build with race detection
make test             # Run Go unit tests with race detection
make test-integration # Run Go integration tests (uses testcontainers, requires Docker)
make lint             # Run golangci-lint (formats code automatically)
make pytest           # Run Python E2E tests
make clean            # Remove binaries and caches
```

Start local MongoDB clusters for testing:

```bash
./hack/rs/run.sh     # Start replica set clusters (rs0, rs1)
./hack/sh/run.sh     # Start sharded clusters
```

**IMPORTANT**: Before running E2E tests or testing any PCSM binary functionality manually, you MUST start the MongoDB containers using the scripts above. The binary requires running source and target MongoDB clusters to function.

Single test:

- `go test -race -run TestName ./package` (unit tests)
- `go test -v -tags integration -run TestName ./pcsm/catalog/...` (integration tests, requires Docker)
- `.venv/bin/pytest tests/test_file.py::test_name` (requires MongoDB containers running; use `poetry run pytest` if poetry works)
- manually execute binary from `./bin` for cases not covered by tests (requires MongoDB containers running)

Cleanup test environments:

```bash
./hack/cleanup.sh       # Clean all environments (rs, sh)
./hack/cleanup.sh rs    # Clean replica sets only
./hack/cleanup.sh sh    # Clean sharded cluster only
```

**IMPORTANT**: Always run `./hack/cleanup.sh` (no arguments) before switching between RS and sharded topologies. Both topologies bind overlapping ports (e.g. 30000), so leftover containers from one topology will cause "port already allocated" errors when starting the other. If cleanup doesn't resolve port conflicts, check for orphaned containers with `docker ps -a`.

## Project-Specific Patterns

### Error Handling

Use `github.com/percona/percona-clustersync-mongodb/errors` (not stdlib):

```go
errors.Wrap(err, "context")   // Returns nil if err is nil
errors.Wrapf(err, "fmt %s", v)
errors.Join(err1, err2)
```

Skip wrapping with `//nolint:wrapcheck` when returning driver errors directly.

### Context Timeouts

Use `github.com/percona/percona-clustersync-mongodb/util`:

```go
util.CtxWithTimeout(ctx, config.Timeout, fn)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [percona/percona-clustersync-mongodb](https://github.com/percona/percona-clustersync-mongodb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
