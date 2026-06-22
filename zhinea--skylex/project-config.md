---
trigger: always_on
description: Skylex is a self-hosted database control plane. The Go backend has two binaries (`skylex-server` and `skylex-agent`) that talk over gRPC/Connect-RPC, plus a Vite + React Router 7 UI in `ui/`.
---

# Skylex — Agent Notes

Skylex is a self-hosted database control plane. The Go backend has two binaries (`skylex-server` and `skylex-agent`) that talk over gRPC/Connect-RPC, plus a Vite + React Router 7 UI in `ui/`.


## Quick start

```bash
# Backend — build both binaries and run the server locally
make build
make dev                                    # starts etcd + minio via docker compose, then runs cmd/server with config.example.yaml
make dev-server                             # runs cmd/server with config.example.yaml (no extra services)

# Full reference stack in Docker Compose
make docker-up                              # builds images and starts server + 3 agents + etcd + minio
make docker-down
```

## Project layout

- `cmd/server` → `skylex-server` binary (control plane)
- `cmd/agent` → `skylex-agent` binary (runs on each DB node)
- `cmd/cli` → empty placeholder for a future `skylexctl`
- `internal/server`, `internal/agent`, `internal/backup`, `internal/db`, `internal/postgres`, `internal/dcs` → internal packages
- `pkg/` → empty; public packages go here if needed
- `proto/skylex/v1/` → protobuf service definitions
- `gen/` → generated Go code from `buf generate` (do not hand-edit)
- `ui/` → Vite + React Router 7 + Tailwind CSS v4 frontend
- `deploy/docker-compose/` → reference deployment and Dockerfiles

## Developer commands

| What | Command |
|------|---------|
| Build both binaries | `make build` |
| Build server only | `make build-server` |
| Build agent only | `make build-agent` |
| Run server locally with dev deps | `make dev` |
| Run server locally (no extra services) | `make run-server ARGS=path/to/config.yaml` or `make dev-server` |
| Run agent locally | `make run-agent ARGS='--server localhost:9090 --token dev-token'` |
| Run all Go tests | `make test` |
| Lint Go code | `make lint` (uses `golangci-lint`; no repo-level config file) |
| Regenerate protobuf | `make proto` (runs `buf lint && buf generate`) |
| Clean build artifacts | `make clean` |
| UI dev server | `cd ui && npm run dev` → `http://localhost:5173` |
| UI typecheck | `cd ui && npm run typecheck` |
| UI production build | `cd ui && npm run build` |

## Configuration

- Server config is YAML; pass the path as the first argument (`./skylex-server config.yaml`). `make dev` uses `config.example.yaml`.
- Settings are merged with `koanf`: YAML file + env vars. Env vars use the prefix `SKYLEX_` and nested keys become `_` (e.g. `SKYLEX_DATABASE_DSN`, `SKYLEX_AUTH_JWT_SECRET`).
- `config.example.yaml` is committed and works as-is for local development.
- Defaults exist for most values; see `internal/server/config.go`. `auth.jwt_secret` defaults to `change-me-in-production` in `config.example.yaml` so dev sessions survive restarts. If `auth.jwt_secret` is left empty, a random secret is generated on startup and a warning is logged; existing JWTs will not validate after a restart.
- Agent settings are layered: CLI flags take precedence over environment variables, which take precedence over the config file (`/etc/skylex/agent.yaml` by default), with `internal/agent/config.go` defaults as the fallback.

```bash
# With flags
make run-agent ARGS='--server localhost:9090 --token dev-token'

# With a token file (recommended for production to avoid leaking secrets in shell history)
make run-agent ARGS='--server localhost:9090 --token-file /etc/skylex/token'
```

- Environment variables are still supported for backwards compatibility and container deployments (`SKYLEX_AGENT_TOKEN`, `SKYLEX_SERVER_ADDR`, `SKYLEX_HOSTNAME`, `SKYLEX_PORT`, `SKYLEX_PG_DATA_DIR`, and `SKYLEX_AGENT_CONFIG`).

## Cluster provisioning workflow

### Node selection
- Cluster creation requires explicit node selection via `node_ids` in `CreateClusterRequest`.
- The first `node_id` becomes the **primary**; all remaining IDs become **replicas**.
- All selected nodes must be unassigned (`cluster_id` empty) and have a linked agent (`agent_id` non-empty).
- `GetByIDs` validates that every supplied ID exists and returns an error listing any missing nodes.

### Service location
- Each cluster has a `service_location` (stored on both the `clusters` and `nodes` tables):
  - `native` — PostgreSQL runs directly on the agent host.
  - `docker` — PostgreSQL runs inside the official `postgres:<version>` container named `skylex-postgres`, with the agent data directory mounted as a persistent volume.
- Agents report `docker_available` at registration; the server logs a warning when Docker is requested but unavailable on a node (it does not block creation).

### Provisioning command sequence
For **native** nodes the server queues only `pg_preflight` initially. The agent reports back:
- `NOTHING_FOUND` → safe to install; server queues `pg_install_native` (if not already installed) then the standard role commands.
- `PG_EXISTS` (existing PostgreSQL or data directory) → node transitions to `installation_state=conflict`; the UI surfaces a per-node conflict card with three choices:
  - **Adopt** — queue `pg_adopt_native` then role init commands (no data loss).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhinea/skylex](https://github.com/zhinea/skylex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
