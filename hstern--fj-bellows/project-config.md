---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repository.
---

# AGENTS.md

Guidance for AI agents (and humans) working in this repository.

## What this is

fj-bellows is a pluggable, ephemeral CI-runner autoscaler for Forgejo Actions.
It polls the Actions job queue, provisions cloud worker VMs, runs ephemeral
per-job runners on them, warm-holds for the paid billing hour, and tears them
down per the provider's billing model. See [README.md](README.md) and
[docs/design.md](docs/design.md).

## Build, test, lint

```sh
make build              # go build ./...
make race               # go test -race ./...  (orchestrator is concurrent — always -race)
make lint               # golangci-lint run ./...  (config in .golangci.yml)
make vuln               # govulncheck ./...
make proto              # buf generate → gen/  (control-plane protobuf + ConnectRPC stubs)
make proto-check        # CI safety: regenerate and fail on drift in gen/
```

The proto targets require `buf`, `protoc-gen-go`, and `protoc-gen-connect-go`
on `$PATH`. Install with `brew install bufbuild/buf/buf` and
`go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
connectrpc.com/connect/cmd/protoc-gen-connect-go@latest`. The `gen/` tree is
committed and CI runs `make proto-check`; if your PR touches `proto/`, run
`make proto` and commit the regenerated `gen/`.

CI runs these in two jobs (`.github/workflows/ci.yml`): `test` (vet, build,
`-race` tests, govulncheck) and `lint` (golangci-lint). Both checks should be
required by branch protection before a PR can merge (configured on the GitHub
repo, not in-tree). `//nolint` directives must name the linter and give a reason
(enforced by `nolintlint`).

## Conventions (please keep)

- **Every package has a `README.md`.** Add one when you add a package.
- **Unit tests for everything.** New behavior comes with tests; keep them fast
  and hermetic (no real network/cloud — use the mocks).
- **Interfaces have hand-written mocks** under a sibling `mock/` package
  (func-field fakes, call recording, concurrency-safe). No codegen tool
  dependency. See `internal/provider/mock` and `internal/orchestrator/mock`.
- **No secrets or deployment specifics in the repo.** No real hostnames, account
  identifiers, usernames, tokens, regions tied to an account, or homelab
  details — in code, tests, docs, or examples. Use generic placeholders. CI
  image names derive from `github.repository` (ghcr) and a secret (Docker Hub),
  not a committed account string.
- Standard library `log/slog` for logging; no logging framework.

## Architecture invariants (don't break these)

- **Billing model is a provider attribute**, not hardcoded. A provider declares
  `BillingModel()`; the core picks the teardown policy
  (`internal/orchestrator/teardown.go`). Per-second → idle timeout; hourly
  round-up → warm-hold + the `:55` rule.
- **The reconcile loop is the single writer of provisioning decisions.**
  Dispatch and teardown goroutines mutate only their own node's state via the
  locked `Pool`. In-flight provisions count as `pending` so concurrent ticks
  don't over-provision.
- **Teardown timers are derived from `Instance.CreatedAt` each tick**, never
  stored, so they survive a restart. Crash recovery rebuilds the pool from
  `provider.List(tag)`.
- **The core never knows provider-specific config.** `provider_config` is an
  opaque `yaml.Node` decoded by the chosen provider (deferred decode).
- **Worker VMs never hold an admin token.** The orchestrator mints the ephemeral
  registration and delivers the one-shot token at dispatch time.
- **Dispatch is mechanism-agnostic and selected per provider.** The `Dispatcher`
  interface takes `(id, addr)`; SSH uses `addr`, a docker-exec dispatcher uses
  `id`. The composition root (`cmd/fj-bellows`) injects it. SSH host keys are
  verified via a per-VM key injected through cloud-init and pre-pinned (with TOFU
  fallback) — don't regress to ignoring host keys.
- **Workers reach Forgejo through the dispatch SSH session — *and so do the
  containers the runner spawns for each step*.** The SSH dispatcher opens a
  reverse port-forward on the dispatch connection and injects a `/etc/hosts`
  override on the worker so the runner process resolves the Forgejo hostname
  to `127.0.0.1`. The dispatcher also writes a forgejo-runner config with
  `container.network: host` and `container.options: --add-host=<host>:127.0.0.1`
  so every job container shares the worker's network namespace (reaching the
  tunnel listener via loopback) and gets the same hosts override. Without the
  runner config, `actions/checkout` and any other step that touches Forgejo
  from inside a containerized step NXDOMAINs even though the runner process
  is happily on the tunnel. Don't drop the runner config; don't reintroduce
  an out-of-process side-car tunnel.
- **The runner config also sets `container.docker_host: automount`** so the
  host docker socket is bind-mounted into every spawned job container. Without
  it, `docker …` steps fail with `no such file or directory`. The worker is
  single-tenant ephemeral and treats its job as trusted (operator's own
  repos), matching every other CI runner stack — same security posture as
  GitHub Actions, GitLab Runner, Drone.
- **The managed cache VM outlives the worker fleet — don't reap it on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hstern/fj-bellows](https://github.com/hstern/fj-bellows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
