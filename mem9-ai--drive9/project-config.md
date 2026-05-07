---
trigger: always_on
description: drive9 is a Go agent-native filesystem — a network drive with semantic search built on top of
---


## Repository overview

drive9 is a Go agent-native filesystem — a network drive with semantic search built on top of
TiDB/MySQL (metadata), S3 (large files), and db9 (small files + embeddings).

Module: `github.com/mem9-ai/drive9`  
Go version: 1.25.1 (see `go.mod`)

---

## Build commands

```bash
make build              # build server + CLI → bin/drive9-server, bin/drive9
make build-server       # server only
make build-cli          # CLI only (supports VERSION= for ldflags)
make build-cli-release  # cross-compile for linux/amd64|arm64, darwin/amd64|arm64
```

Direct Go equivalents:

```bash
go build -o bin/drive9-server ./cmd/drive9-server
go build -o bin/drive9 ./cmd/drive9
```

All binaries are built with `CGO_ENABLED=0`.

---

## Test commands

```bash
# full suite
make test
# single package
make test TEST_PKGS='./pkg/datastore/...'
# single test
make test TEST_RUN='TestInsertAndGetNode' TEST_PKGS='./pkg/datastore/...'
```

MySQL-backed tests require a container runtime or an explicit DSN:

```bash
# Use an existing MySQL/TiDB instance
DRIVE9_TEST_MYSQL_DSN='user:pass@tcp(127.0.0.1:3306)/drive9_test?parseTime=true' make test
```

If `DRIVE9_TEST_MYSQL_DSN` is unset and `podman` is available, `make test` auto-configures
testcontainers via `scripts/test-podman.sh`. Otherwise a Docker-compatible runtime is used.

If a direct `go test` run fails with `rootless Docker not found`, retry with `make test`
so the project can use `scripts/test-podman.sh` to route testcontainers through Podman.

**E2E smoke tests** (not `go test`) live in `e2e/` and target live deployments.
Read `e2e/AGENTS.md` first for endpoint selection, `drive9-server-local` workflow,
environment variables, script coverage, and known expectations.

Common entry points:

```bash
DRIVE9_BASE=https://... bash e2e/api-smoke-test.sh
DRIVE9_BASE=https://... bash e2e/cli-smoke-test.sh
DRIVE9_BASE=https://... bash e2e/smoke-all.sh
```

---

## Lint and format

```bash
make lint            # golangci-lint run (v2.5.0, installed to bin/)
make fmt             # golangci-lint run --fix
```

golangci-lint is auto-installed to `bin/golangci-lint` on first `make lint`. There is no
`.golangci.yml`; linter runs with default settings. CI (`code-ci.yml`) enforces lint before
tests on every PR to `main`.

---

## Local dev server

```bash
source ./scripts/drive9-server-local-env.sh
export DRIVE9_LOCAL_INIT_SCHEMA=true   # only for a fresh/disposable database
make run-server-local
```

The env script sets defaults for `DRIVE9_LOCAL_DSN`, local mock S3, and Ollama-compatible
embedding. Override any var before running.

---

## Project layout

```
cmd/drive9/             CLI entrypoint (cp, cat, ls, mv, rm, mount, umount, ...)
cmd/drive9-server/      Server entrypoint
.github/ISSUE_TEMPLATE/ GitHub issue templates (bug report / enhancement / feature request)
pkg/
  backend/              AGFS FileSystem implementation (Drive9Backend)
  client/               Go SDK HTTP client
  datastore/            Core metadata store (TiDB/MySQL)
  embedding/            Embedding provider integration
  encrypt/              Encryption helpers
  fuse/                 FUSE mount (go-fuse/v2)
  logger/               Structured logging (zap)
  meta/                 Metadata/search models
  metrics/              Metrics recording
  s3client/             S3 interface (AWS + local mock)
  server/               HTTP server (/v1/fs/{path} router)
  tenant/               Tenant schema management
  pathutil/             Path canonicalization and validation
  semantic/             Durable background task types
  traceid/              Trace ID helpers
internal/
  testmysql/            MySQL test helpers (shared across packages)
e2e/                    Live bash smoke tests; read e2e/AGENTS.md first
scripts/                Shell helpers for local dev and test
docs/                   Design documents
site/                   Frontend / release assets
```

When creating a new GitHub issue, follow the templates under `.github/ISSUE_TEMPLATE/`
to keep issue structure and required context consistent.

### CLI find tag semantics

- `drive9 fs find ... -tag key=value` is an exact key/value match.
- `drive9 fs find ... -tag key` means tag-key existence match.
- `-tag` does not support fuzzy, prefix, contains, or regex matching.

---

## Code style guidelines

### Package structure

- One package per directory; package name matches directory name.
- Package-level doc comment on the first file: `// Package foo provides ...`
- Each package has a focused responsibility — avoid cross-cutting concerns.

### Imports

Group imports in three blocks separated by blank lines:

```go
import (
    "context"         // stdlib
    "fmt"

    "go.uber.org/zap" // third-party

    "github.com/mem9-ai/drive9/pkg/logger" // internal
)
```

Use an import alias only when there is a naming collision:

```go
pathpkg "path"  // disambiguates from a local "path" variable
```

### Naming conventions

- Packages: short, lowercase, no underscores (`datastore`, `pathutil`, `s3client`).
- Exported types/functions/consts: PascalCase (`FileNode`, `StorageType`, `RRFMerge`).
- Unexported: camelCase (`smallFileThreshold`, `newBaseBackend`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mem9-ai/drive9](https://github.com/mem9-ai/drive9) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
