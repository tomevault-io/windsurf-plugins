---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

acompose runs an existing `docker-compose.yml` on Apple's `container` CLI (the macOS-native, VM-per-container runtime, which ships with no Compose support). It is a single Go binary; all sources live in `src/`: `main.go` (CLI + orchestration), `ui.go` (the `acompose ui` live dashboard), `main_test.go` (unit tests). `src/prototype/acompose.py` is a superseded Python prototype kept for history — don't extend it.

## Commands

```sh
go build -o acompose ./src  # build (or: make build)
make darwin                 # release-style darwin/arm64 binary
make fmt                    # gofmt -w src (CI fails on unformatted code)
make vet                    # go vet
make test                   # go test ./src (pure unit tests, no `container` exec)
make dryrun                 # build + run `acompose up --dry-run` against examples/
make check                  # fmt + vet + test + dryrun
```

Unit tests cover the pure helpers (toposort, extractIPv4, runCmd/buildCmd construction, volName). The integration test lives in `.github/workflows/ci.yml`: it runs `acompose up --dry-run` and `down --dry-run` in `examples/` and asserts on the printed `container` commands (start order, env interpolation, `--publish` flags, build translation, loud warnings, reverse teardown order). If you change command construction or output text, update those assertions.

Real (non-dry-run) execution requires macOS 26 + Apple's `container` CLI on Apple Silicon, but `--dry-run` works anywhere — that's the primary development loop.

## Architecture

The pipeline in `cmdUp` (src/main.go):

1. **Parse** — all compose-spec handling (interpolation, `.env`, `env_file`, overrides, profiles) is delegated to the official `compose-spec/compose-go` library via `loadProject`. Never hand-parse compose semantics.
2. **Order** — `toposort` walks `depends_on` (cycles are fatal; names sorted for deterministic order).
3. **Warn** — `warnUnsupported` is the catalog of platform gaps (exec healthchecks, `deploy:`, `entrypoint:`, `user:`, x86 images; `restart:` points at `acompose watch`). Design rule: anything the platform can't honour must be warned about loudly, never silently dropped.
4. **Translate & run** — every `container` subcommand is constructed in one place: `ctr` / `buildCmd` / `runCmd` / `hostsInjectCmd` near the top of src/main.go. The `container` CLI is young and renames flags between releases; keeping construction centralized makes that a one-line fix. Don't scatter `exec.Command("container", ...)` elsewhere (the UI's read-only polling in ui.go is the lone exception). Named volumes are real: `container volume create` before start, `--volume <name>:<target>` mounts, deletion on `down -v` (`volName`/`namedVolumes`; compose-go pre-fills `VolumeConfig.Name` as `<project>_<key>`).
5. **Wire networking** — each container's real IP (from `container inspect`, parsed by `extractIPv4` — a key-aware walk that skips `gateway`/`dns` keys and prefers `address` keys, because the JSON schema shifts and the first IPv4 found may be the gateway) is written into every peer's `/etc/hosts` immediately and bidirectionally, so `db:5432` works in unmodified apps. `<SERVICE>_HOST` env vars are always injected as the fallback for shell-less or non-root images. `condition: service_healthy` is approximated by TCP-polling the dependency's first published port (`waitTCP`). After starting, `cmdUp` verifies actual container state via `container ls --all` and reports per-service running/stopped in the summary.

Beyond up/down:

- `cmdWatch` — supervisor loop honouring `restart:` policies (the runtime has none): polls, `container start`s exited services, and calls `rewireAll` (shared with `cmdRefresh`) to clean stale /etc/hosts entries and re-inject fresh IPs. Refuses `--dry-run`.
- `cmdUpdate` — pulls images and recreates only services whose manifest digest moved (`imageDigests` extracts only `"digest":` keys — matching every sha256 would catch layer blobs and false-positive after `run`'s shallower implicit pull); `build:` services are rebuilt. Ends with `rewireAll` when anything changed.
- `acompose refresh` re-reads IPs and rewrites hosts entries after sleep/wake. `cmdDown` tears down in reverse topological order. `stats` is a passthrough to `container stats` with project cnames.
- src/check.go — `acompose check`: no-runtime compatibility report. `analyzeService` is the single source of truth for platform-gap findings; `warnUnsupported` (main.go) is a thin renderer over it — change classifications there, never in two places. Exit 1 on blockers.
- src/doctor.go — `acompose doctor`: environment readiness; pure verdict functions + injectable `doctorEnv` for hermetic tests; dispatched before loadProject (works with no compose file). `testedContainerVersion` const is the compat-matrix anchor.
- src/dns.go — `acompose dns setup|status|teardown` wraps `container system dns` (needs sudo once; on permission failure prints the exact sudo command). cmdUp prints a dim `host DNS:` line when the project domain exists (probe skipped in dry-run). What resolves is `<container-name>.<project>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [htlin222/acompose](https://github.com/htlin222/acompose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
