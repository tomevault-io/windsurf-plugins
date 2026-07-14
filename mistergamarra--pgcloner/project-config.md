---
trigger: always_on
description: Guidance for working in this repo.
---

# CLAUDE.md

Guidance for working in this repo.

## What this is

A general-purpose Go CLI: `urfave/cli/v3` command tree, `koanf`-based env
config layered with CLI flags, one `internal/` package per concern. The
binary's entry point lives at `cmd/pgcloner/main.go` (standard Go
project layout — `internal/` packages, not `main.go`, hold all the logic).

Nothing in this repo assumes a specific Teleport cluster, DB role naming
scheme, or Docker image — see `internal/config` and `cmd/pgcloner/main.go`
for the full set of flags/env vars, all with vendor-neutral defaults (or
no default, where a value must be supplied).

Module path is `github.com/mistergamarra/pgcloner` — it must
match the actual GitHub repo path, since `go install
github.com/mistergamarra/pgcloner/cmd/pgcloner@latest`
(the README's primary install method) only resolves correctly when it
does. If this repo is ever forked/renamed, update `module` in `go.mod`,
every `github.com/mistergamarra/pgcloner/internal/...` import,
`.goreleaser.yaml`'s `release.github.owner`/`name`, and the README's
install commands and badge URLs together — they'll silently drift
otherwise.

## Conventions

- **CLI framework**: `github.com/urfave/cli/v3`, `Suggest: true` on the
  root command. Global flags mirror every `config.AppConf` field so users
  never have to look up an env var name to override a setting.
- **Config**: `koanf` loaded from `PGCLONER_*` env vars via
  `internal/config`, `.env` auto-loaded next to the binary via `godotenv`.
  CLI flags are layered on top in `main.applyFlagOverrides` and always win.
- **Errors**: every command returns an `error` from its `Action` (never
  calls `os.Exit` itself); `main` prints it directly to stderr with
  `fmt.Fprintln` — not `log/slog` — since these messages are read by a
  human at a terminal, and slog's text handler escapes newlines (breaking
  `doctor`'s multi-line missing-tool hints).
- **Preflight checks**: `internal/doctor` knows which external binary each
  command needs (`doctor.Binaries[].UsedBy`). Every command that shells
  out gets a `Before: requireTools("<name>")` hook in `main.go`, so a
  missing `tsh`/`pg_dump`/`psql`/`docker` fails immediately instead of an
  `exec: not found` mid-wizard. `doctor` itself has no such hook — it's
  the one command allowed to report missing tools rather than fail on
  them.
- **Install instructions live in exactly one place**: the README's
  Prerequisites table. `doctor.go` deliberately has no per-tool install
  command (no `brew install ...`, no OS detection) — every missing-tool
  message (`doctor.readmePointer`) just says "see the Prerequisites
  section in README.md." Keeping install commands in code risks two
  copies drifting out of sync (wrong Homebrew formula name, missing an
  OS, etc.); if the required tools or how to install them change, update
  the README table — `doctor.go` doesn't need touching.

## Deviations from the bash scripts (and why)

- **Interactive selection**: `github.com/charmbracelet/huh` replaces the
  bash scripts' hand-rolled numbered-menu + `fzf` combo (`internal/uiselect`).
  This drops the `fzf` binary dependency entirely — `huh`'s multi-select
  already supports filtering and toggle-all.
- **Postgres queries**: `internal/pgutil` uses `jackc/pgx/v5` directly for
  the interactive lookups (list databases/schemas/tables), instead of
  shelling out to `psql -t -A`. `pg_dump`/`psql` are still invoked via
  `os/exec` for the actual dump and restore — those remain the
  authoritative tools for moving data.
- **Docker**: `internal/dockerutil` shells out to the `docker` CLI rather
  than the Docker SDK, matching the original scripts' approach and keeping
  the dependency surface small.

## Key behaviors to preserve when touching this code

- **`127.0.0.1`, never `localhost`**: macOS resolves `localhost` to `::1`
  first; `tsh proxy db` and Docker's published port both bind IPv4.
- **Proxy readiness**: `teleport.Proxy.Wait` does a raw TCP dial, not a
  Postgres ping — Teleport's tunnel needs a username+DB before it speaks
  the PG protocol, so a `pg_isready`/pgx ping fails against the bare tunnel
  before `db login` completes.
- **Container readiness is the opposite case**: `pgutil.WaitReady`
  (used by `restorecmd` after starting a fresh container) retries a real
  Postgres ping, not a raw TCP dial — here the container's own port *is*
  reachable early, since the official `postgres` image briefly runs a
  temporary internal server (bound to the same port) to execute init
  scripts on first run, then restarts into its real listening process. A
  bare TCP check can succeed against that temporary server and then get
  "connection reset by peer" on the actual protocol handshake moments
  later; retrying the real ping rides out the restart instead of racing
  it once. Don't swap this back to a TCP-only check.
- **Permission-denied retry**: `dumpcmd.runDump` parses `permission denied
  for table <name>` specifically (not `LOCK TABLE`, which lists every
  table being dumped) and retries up to 5 times, excluding one more table
  each attempt. This is how dumps succeed even when the connecting DB user
  only has read access to some tables.
- **`--exclude-table` blacklist, not `--table` whitelist**: a whitelist
  drops shared types, sequences, and extensions from the dump.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mistergamarra/pgcloner](https://github.com/mistergamarra/pgcloner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
