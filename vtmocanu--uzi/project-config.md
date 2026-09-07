---
trigger: always_on
description: "Uzinele Întunecate" (uzi): an AI dark factory. Go API + React SPA + PostgreSQL + an opt-in per-user worker container, run via docker-compose on a laptop. Users connect a forge and an Anthropic token; agents work `PRD`-labeled issues end to end (plan → approval gate → implement ⇄ review → branch + MR, never touching `main`).
---

## What this is

"Uzinele Întunecate" (uzi): an AI dark factory. Go API + React SPA + PostgreSQL + an opt-in per-user worker container, run via docker-compose on a laptop. Users connect a forge and an Anthropic token; agents work `PRD`-labeled issues end to end (plan → approval gate → implement ⇄ review → branch + MR, never touching `main`).

## Destructive operations

Always loaded: a path-scoped rule fires on a file READ, i.e. after the decision to act, and everything here is irreversible.

- 🔴 **`docker compose -p uzi down -v`, from any directory holding a compose file, destroys real data.** It removes `uzi_pgdata` and `uzi_agentdata`, which carry the real admin and forge data. `cd main && docker compose -p uzi up` brings the stack back; the volumes do not come back.
- **Never pass `-p uzi` to a `down`, and never add `-v` to one.**
- **Never `docker compose down` from a worktree.** Belt-and-braces today: the recorded `config_files` path no longer exists (see the `.env` note in `.claude/rules/stack.md`), so discovery cannot reach project `uzi` and a bare `down` resolves the worktree's own project. Re-creating that file restores the hazard.
- 🔴 **Never glob `uzi-` when tearing down containers.** The dev stack (`uzi-web-1`, `uzi-api-1`, `uzi-agent-1`, `uzi-db-1`) shares a daemon with throwaway test containers, and `uzi-db-1` shares `postgres:17` with them, so neither `--filter name=uzi-` nor `--filter ancestor=postgres:17` can tell them apart:

```
uzi-seam5b-pg    postgres:17   Up 52 seconds   <- throwaway
uzi-final-95941  postgres:17   Up 5 minutes    <- throwaway
uzi-db-1         postgres:17   Up 2 weeks      <- the REAL database
```

1. **Name throwaways OUTSIDE the `uzi-` namespace** (`cdr-*`, `aud-*`, `vm-rev-*`). Load-bearing: it removes the failure mode instead of relying on discipline.
2. **Tear down only your own container, by exact name.** Never a `uzi-*` glob, never `docker compose down` from a worktree.
3. **If you see a container you did not create, leave it.** Same for processes: a stray `run-e2e.sh` or `run-store-it.sh` may belong to another session, and refusing to kill an unowned process is correct, not obstructive.
   - Attribute a process by the **redirected log path alone**, and only when its name is distinctive. Shell-snapshot path is per-CLI-session, not per-agent; cwd is shared across agents in one worktree. Both manufacture a confident false match (measured 2026-08-02).
   - **If you cannot attribute a process, leave it.**

`./e2e/run-store-it.sh` names its container `uzi-store-it-$$`, inside the namespace rule 1 avoids. It is PID-unique and tears itself down by exact name, so it is safe to run; rule 2 holds the line there.

## Commands

Install the pinned `task`, which every command below needs:

```sh
go install github.com/go-task/task/v3/cmd/task@v3.51.1   # pinned, sumdb-verified
```

- Version-matched to CI. `brew install go-task` works but is unpinned and drifts from CI.
- `task` does **not** go in `devbox.json`: that is tier-2 *worker* config whose `packages` array is provisioned into opted-in runs (`agent/src/repo-tools.ts`), not a contributor environment.
- `go install` builds from source, so the binary is not byte-identical to the release tarball CI's `.task_setup` sha256-verifies. Matching `task --version` is the equivalence check, as for `sqlc@v1.31.1`.

### Gate targets

- **`Taskfile.yml` at the repo root is the only place a gate recipe is written.** `task --list` enumerates them.
- `task gate` runs everything, `gate:repo` first: the checks with no component of their own (shellcheck, yamllint, GitHub Actions lint incl. embedded scripts, the Homebrew formula, migration numbering and additivity, spec numbering, no-binary-text, `scan:secrets` with gitleaks, `sast:semgrep`). `task --dry gate:repo` lists them. `gate:repo` / `gate:api` / `gate:controller` / `gate:web` / `gate:agent` run one slice each.
- `.github/workflows/ci.yml` invokes the same targets, mostly per-toolchain (`validate:*`, `lint:api`, `lint:controller`, `test:*`) plus repo-wide `lint:repo`, so local and CI cannot drift. `test:api-store-it` invokes none by design: its ran/skipped assertion is CI-specific.
- Every load-bearing flag lives in `Taskfile.yml` with its reason. Task echoes each command, so `-race`, `-count=1` and `--test-timeout=120000` stay visible; that echo is how you notice one going missing.
- **Component gates run serially, deliberately.** CPU contention is a measured flake source, and interleaved output defeats reading the named failing test.
- `task gate:api` carries `-race`, which the hand-typed `go test -count=1 ./...` it replaced did not and CI always did. It runs longer (51.8s, measured 2026-08-02) and can redden on a real data race.
- Read the `test:api` comment in `Taskfile.yml` for `-race` / `-count=1` provenance, not a commit cited from memory.
- Read `web/vite.config.ts` for its `testTimeout` rather than trusting a quoted figure: the suite-wide value is 20000 and two tests carry a per-test cap of 120000.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vtmocanu/uzi](https://github.com/vtmocanu/uzi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
