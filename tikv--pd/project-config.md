---
trigger: always_on
description: - Scope: Governs the entire repository. If another `AGENTS.md` exists deeper, it overrides in its subtree.
---

# PD Agent Guide

- Scope: Governs the entire repository. If another `AGENTS.md` exists deeper, it overrides in its subtree.
- Cursor rules: none in `.cursor/rules/` or `.cursorrules`.
- Copilot rules: see `.github/copilot-instructions.md`.
- Purpose: Give human and agent contributors a concise, reliable playbook.

## Quick Facts
- Language: Go modules (root + `client/` submodule).
- Go version: CI uses 1.25 (install >=1.25).
- Main binaries: `pd-server`, `pd-ctl`, `pd-recover`, tool suite.

## Build Shortcuts
- Default all: `make build` (pd-server, pd-ctl, pd-recover).
- Full dev loop: `make dev` (build + check + tools + test).
- Lightweight: `make dev-basic` (build + check + basic-test).
- No dashboard: `make pd-server-basic` (sets `SWAGGER=0 DASHBOARD=0`).
- With swagger spec: `SWAGGER=1 make build` (runs `swagger-spec`).
- With custom dashboard distro: set `DASHBOARD_DISTRIBUTION_DIR` then `make build`.
- Specific tool: `make pd-ctl`, `make pd-tso-bench`, etc.
- Race build: `WITH_RACE=1 make build` (CGO on, `-race`).
- FIPS/boringcrypto: `ENABLE_FIPS=1 make build`.
- Simulator: `make simulator`.
- Docker image: `make docker-image` (needs Docker daemon).

## Client Module (client/)
- Default pipeline: from `client/`, run `make` (static + tidy + test).
- Tests with race/tags: `make test` (deadlock tag, race, cover; auto failpoints).
- Fast tests: `make basic-test`.
- CI coverage: `make ci-test-job`.
- Lint/static: `make static` (gofmt, golangci-lint, leakcheck).

## Lint & Static Analysis
- Primary entry: `make check` (tidy + static + generate-errdoc).
- Static combo: `make static` runs gofmt -s, golangci-lint, leakcheck (PACKAGE_DIRECTORIES/SUBMODULES respected).
- Tidy: `make tidy` must leave `go.mod`/`go.sum` clean (CI enforces empty diff).
- Error docs: `make generate-errdoc` (updates `errors.toml`).
- golangci-lint config: see `.golangci.yml` (gofmt/goimports/gci formatters; depguard bans `github.com/pkg/errors`, `go.uber.org/atomic`, `math/rand`; prefers `math/rand/v2`; goheader copyright; revive + testifylint extensive; waitgroup-by-value forbidden).
- leakcheck excludes `tests/server/join/join_test.go`.
- Formatter order (gci): standard, default, `prefix(github.com/pingcap)`, `prefix(github.com/tikv/pd)`, blank.

## Test Matrix
- Full suite: `make test` (tags deadlock, CGO=1, race, cover; auto failpoints).
- Basic fast: `make basic-test` (no tests/ packages, no race; failpoints enabled).
- Targeted (single pkg/test): `make gotest GOTEST_ARGS='./pkg/foo -run TestBar -count=1'` (auto failpoints).
- UT binary: `make ut` -> `./bin/pd-ut run --ignore tests --race --junitfile ./junitfile`.
- CI shard: `make ci-test-job JOB_INDEX=N` (needs dashboard-ui + pd-ut built).
- TSO function: `make test-tso-function` (tags `without_dashboard,deadlock`, race on).
- Real cluster: `make test-real-cluster` (uses `tests/integrations/realcluster`; wipes `~/.tiup/data/pd_real_cluster_test`).
- Coverage split: `make test-with-cover-parallel` after `make split`.
- Clean test artifacts: `make clean-test` (clears `/tmp/pd_tests*`, go test cache, UT bins, playground log).

## Failpoints Discipline
- PD uses `github.com/pingcap/failpoint` to inject test-only branches, returns, and pauses into specific code paths.
- Keep failpoints enabled only for tests; disable immediately after (`make failpoint-disable` or `make clean-test`) to avoid polluting the codebase.
- Prefer make targets that auto-enable/disable failpoints (recommended: `make gotest ...`, `make test`, `make basic-test`).
- If you must run `go test` manually, use this rule:
  - Target uses failpoints (for example imports `github.com/pingcap/failpoint`): `make failpoint-enable` -> `go test ...` -> `make failpoint-disable`.
  - Target does not use failpoints: run `go test ...` directly.
- Runtime model:
  - `failpoint.Enable` / `failpoint.Disable` only change runtime evaluation state.
  - A failpoint takes effect only when execution reaches the injected site again.
  - Enabling a failpoint does not replay startup or initialization logic that has already finished.
- If failpoint semantics are unclear, inspect the injected code path and nearby tests before changing test flow or assertions.
- Useful lookup pattern: `rg -n "failpoint.Inject|failpoint.InjectCall|failpoint.Enable" pkg tests server`.
- If that is still not enough, read the upstream `README.md` in `github.com/pingcap/failpoint` before changing the test design.
- When tests need controllable fault injection, compare `failpoint` and `mock` first; prefer the simpler and more maintainable option instead of defaulting to `mock`.
- Never edit code or run non-test commands while failpoints are enabled. If unsure about state, run `make failpoint-disable` before continuing.
- Never commit generated failpoint files or leave failpoints enabled; verify `git status` is clean before pushing.
- If failpoint-related tests misbehave, rerun after `make failpoint-disable && make failpoint-enable` to ensure a clean state.

## Imports & Formatting
- Use gci/goimports ordering: stdlib | third-party | pingcap | tikv/pd | blank.
- Avoid dot-imports; revive rule will warn.
- Keep imports deduped; run gofmt/goimports/gci.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tikv/pd](https://github.com/tikv/pd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
