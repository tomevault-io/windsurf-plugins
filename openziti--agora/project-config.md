---
trigger: always_on
description: Agora is a native zero-trust overlay network for agent-to-agent communication, built on OpenZiti. It provides secure connectivity primitives at Layer 1 (Network) and governed agent collaboration services at Layer 2 (Collaboration).
---

# Coding Agent Instructions

## Agora

Agora is a native zero-trust overlay network for agent-to-agent communication, built on OpenZiti. It provides secure connectivity primitives at Layer 1 (Network) and governed agent collaboration services at Layer 2 (Collaboration).

This repository is in early-stage development. Favor simple, explicit structure and keep implementation aligned with `docs/current/architecture/overview.md` and the relevant layer docs under `docs/current/`.

If you've never worked in this repo before, read [`docs/current/maintainers/onboarding.md`](./docs/current/maintainers/onboarding.md) once — it covers what runs where (dev controller `:18081` vs demo controller `:18080`, external Postgres + OpenZiti), the two ways to bring the stack up (`bin/demo-up.sh` standalone vs `--attach` against your own controller), and the operational footguns below in long form.

## Where to look

| If you're changing… | Look in |
|---|---|
| Controller HTTP behavior | `internal/controller/<operationId>.go` (one file per OpenAPI operation) |
| API contract | `internal/api/specs/` (canonical) → regenerate with `./bin/generate_rest.sh` |
| DB schema | `internal/persistence/migrations/NNNN_<name>.sql` + repo in `internal/persistence/<table>.go` |
| Auth / principal middleware | `internal/controller/auth_middleware.go`, `internal/controller/service.go` |
| CLI command | `cmd/agora/<group><Verb>.go` (registered via `init()` in the group file) |
| Agent SDK | `sdk/agent/` |
| Network runtime | `internal/network/tunnelruntime/`, `internal/network/daemon/` |
| Dashboard screens | `ui/src/screens/<Screen>.tsx` |
| Dashboard API client | `ui/src/lib/api/<resource>.ts` (hand-written; re-exported from `index.ts`) |
| Dashboard polling | `ui/src/lib/api/hooks.ts` — `useApiResource(load, { intervalMs })` is the blessed live-refresh pattern |
| Demo content | `cmd/demo-bootstrap/topology.yaml` + handlers under `cmd/demo-bootstrap/` |
| Demo workers | `examples/macro-pulse/cmd/macro-pulse-*` |

## Documentation Structure

- `docs/` is split into `docs/current/` (verifiable against running code) and `docs/future/` (vision, work orders, deferred designs). New docs default to the bucket that matches their nature; promote from `future/` to `current/` as work lands. See [`docs/README.md`](./docs/README.md) for the full convention — what each bucket means, where new docs go, and the promotion procedure.
- The canonical architecture, status, maintainer, and roadmap materials live under `docs/`.
- Use `docs/current/architecture/overview.md` for cross-layer architecture and system-wide design.
- Use `docs/current/layer-1/spec.md`, `docs/current/layer-1/status.md`, and `docs/current/layer-1/agent.md` for Layer 1 (Network) normative behavior, current state, and local-runtime design.
- Use `docs/current/layer-2/spec.md` and `docs/current/layer-2/status.md` for Layer 2 (Collaboration) design and implementation status.
- Use `docs/current/maintainers/current-state.md` for repo-shape, workflow, and maintainer-facing current-state context.
- Use `docs/future/roadmap/post-mvp.md` for explicitly deferred work such as metrics and limits.
- Do not create new root-level planning or handoff docs that duplicate the `docs/` canon.
- Keep spec docs normative, status docs factual/current-state, and roadmap docs limited to deferred or later-phase work.

## Development Commands

### Build Commands
- Full build: `go build ./...`
- Install binary: `go install ./...`

### Build verification (do not leave stray binaries in the working tree)
- `go build ./...` at the repo root is safe: it only compiles and never writes binaries to disk.
- `go install ./...` is safe: outputs land in `$GOBIN` / `$GOPATH/bin`, not in the working tree.
- `go vet ./...` is safe and is the preferred compile-check for a single change.
- **Never** run `go build ./path/to/cmd` or `go build ./path/to/cmd/...` — those forms emit the built binary into the current directory, which is almost always the repo root for an agent session. Those stray binaries are trivial to accidentally `git add` and commit (and at ~50 MB each, they are expensive mistakes).
- If a specific `main` package needs to be compiled to check a change, use `go build -o /dev/null ./path/to/cmd` so no file is written.
- `.gitignore` already lists the known `main` package output names as a safety net, but the rule above is what prevents the problem in the first place.

### Testing
- Go tests: `go test ./...`
- Persistence integration tests use PostgreSQL containers via `testcontainers-go`
- Before finishing a change, run the narrowest relevant tests first, then `go test ./...` when the change touches shared code or project wiring

### Known Operational Footguns

Failure modes that aren't obvious from the code. Long-form descriptions in [`docs/current/maintainers/onboarding.md`](./docs/current/maintainers/onboarding.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openziti/agora](https://github.com/openziti/agora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
