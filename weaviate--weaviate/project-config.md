---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Weaviate is an open-source, cloud-native vector database written in Go. It stores both objects and vectors, supporting semantic search, hybrid search (BM25 + vector), RAG, and reranking.

## No bug is ever out of scope

This is a production database. Data loss and silent failures are unacceptable, full stop. If you uncover or even *suspect* a bug — adjacent failure mode, race window, edge case in a related journey, anything — you MUST address it in the same change set. Acceptable outcomes:

1. **Reproduce and fix it.** Include a regression test that fails without the fix and passes with it.
2. **Reproduce it and commit a failing (red) test** that pins the bug, then escalate explicitly to the user. Never silently leave a known-bad code path with no test.

Unacceptable:

- "Out of scope." There is no out of scope for bugs in this codebase. If you find one, you own it until it's either fixed or pinned with a failing test.
- "Known issue, leaving for follow-up." Same rule — pin it with a failing test before you stop working on it.
- Fixing only the one specific reproduction a reviewer gave you and shipping. If a bug exists in journey X, enumerate every realistic adjacent journey (X-1, X+1, multi-property, multi-round, every related state-machine transition) and add tests for the lot. Whack-a-mole on individual repros is forbidden.

When you find a bug while working on something else, the response is: stop the original work, switch context to the bug, write the test, write the fix (or commit the red test and surface it loudly), then resume. The original task can wait.

This rule overrides any conflicting guidance about staying focused, minimal diffs, or scope discipline. Bug coverage wins.

## Build & Run

```bash
# Build the binary (CGO_ENABLED=0, static linking)
make weaviate

# Build debug binary (with delve support)
make weaviate-debug

# Run locally (starts dependencies via docker-compose)
make local

# Build Docker image
make weaviate-image

# Regenerate gRPC protobuf code
make grpc

# Regenerate mocks (uses mockery via Docker)
make mocks
```

## Testing

When creating tests prefer table-driven tests.

### Unit Tests
```bash
# Run a single package's tests
go test ./adapters/repos/db/lsmkv/...

# Run a specific test
go test -run TestBucketReplace ./adapters/repos/db/lsmkv/

# Run all unit tests (slow, use sparingly)
go test -race -count 1 $(go list ./... | grep -v 'test/acceptance' | grep -v 'test/modules')
```

### Integration Tests
Use build tag `integrationTest`. Run per-package only, never repo-wide:
```bash
go test -tags integrationTest -count 1 -race ./adapters/repos/db/...
```

### E2E / Acceptance Tests
Prefer testcontainers (modern style) over requiring a running Weaviate instance (legacy style). Never run the full e2e suite. Run only specific packages you changed:
```bash
go test -count 1 -race -timeout 15m ./test/acceptance/grpc/...
```
When adding new e2e tests, prefer creating a separate package. Only extend existing packages when tests clearly fit.

**Pre-building the Docker image for acceptance tests:**
By default, testcontainers builds the Weaviate Docker image from source on every test run. For packages with many test functions (e.g. `reindex_multinode` with 9 tests), this rebuilds the image 9 times, wasting disk and time.

Pre-build once and reuse:
```bash
# Build the image once (tag it with a recognizable name)
make weaviate-image WEAVIATE_IMAGE=weaviate-test:local

# Run tests with the pre-built image (skips docker build entirely)
TEST_WEAVIATE_IMAGE=weaviate-test:local go test -count 1 -race -timeout 20m ./test/acceptance/reindex_multinode/...
```

Always rebuild the image after code changes. The `TEST_WEAVIATE_IMAGE` env var is read by `test/docker/compose.go` and applies to all testcontainer-based tests.

**Adding new acceptance test CI jobs (`test/run.sh`):**
When adding a new `run_acceptance_*` function in `test/run.sh`, it **must** build the Docker image and export `TEST_WEAVIATE_IMAGE` before running tests. Without this, testcontainers builds from source on every test function, which is slow and can exceed startup timeouts. Follow this pattern:
```bash
function run_acceptance_my_new_tests() {
  echo_green "acceptance — my-tests: building weaviate/test-server image..."
  GIT_REVISION=$(git rev-parse --short HEAD)
  GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
  docker compose -f docker-compose-test.yml build \
    --build-arg GIT_REVISION="$GIT_REVISION" \
    --build-arg GIT_BRANCH="$GIT_BRANCH" \
    --build-arg EXTRA_BUILD_ARGS="-race" \
    weaviate
  export TEST_WEAVIATE_IMAGE=weaviate/test-server
  run_aof_group "my-tests" test/acceptance/my_tests
}
```

### Linting
Always validate linters pass at the end of a task:
```bash
golangci-lint run ./...
./tools/linter_go_routines.sh
```

The goroutine linter enforces that all goroutines use `entities/errors/go_wrapper.go` instead of bare `go` statements.

## Architecture

The codebase follows a **hexagonal (ports-and-adapters) architecture**:

```
cmd/weaviate-server/     → Entry point (go-swagger generated main)
adapters/
  handlers/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weaviate/weaviate](https://github.com/weaviate/weaviate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
