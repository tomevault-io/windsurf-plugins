---
trigger: always_on
description: A Go rewrite of `soularr`: a bridge between Lidarr and Soulseek. It polls Lidarr for
---

# slusk

A Go rewrite of `soularr`: a bridge between Lidarr and Soulseek. It polls Lidarr for
wanted albums, searches Soulseek, downloads candidates, and hands finished albums back
to Lidarr for import. Unlike soularr it keeps persistent state, so a restart does not
strand in-flight downloads.

Go 1.26.3. React 19 + TypeScript in `web/`. Postgres only — `internal/store` opens `pgx`
and nothing else, and the migration runner takes a `pg_advisory_lock`. SQLite survives
solely in `cmd/sqlite2pg`, the one-off tool that reads a legacy SQLite file and writes it
into Postgres.

## Build and test

```bash
make build     # builds the web UI, then the Go binary
make ui        # web UI only (npm ci + vite build → internal/observ/web/dist)
make test      # go test ./... && npm test
make dev       # vite dev server
go test ./...            # run this before claiming anything works
go test ./... -race      # required for anything touching concurrency
```

`go vet ./...` and `gofmt -l .` should both be clean.

## Merging to main deploys to the canary

This is the single most important thing to know about this repo.

`.gitea/workflows/release.yml` runs on every push to `main`, reads conventional commit
prefixes since the last tag, and pushes a new `v*` tag. That tag triggers `deploy.yml`,
which builds the image, publishes it as `:vX.Y.Z` and `:edge`, and tells the homelab
updater to redeploy **the maintainer's own instance**.

| Prefix | Effect |
|---|---|
| `feat:` | minor bump → **live on the canary within minutes** |
| `fix:` | patch bump → **live on the canary within minutes** |
| `!:` or `BREAKING CHANGE` | major bump → live on the canary |
| `chore:`, `docs:`, `ci:`, `refactor:`, `style:`, `test:` | no bump, no build |

There is still no staging step, and the canary is not a test rig: it runs a real music
library against real Soulseek accounts, and it is deliberately allowed to break. "Merge
it and see" remains a production action — just a production of one.

Other people's instances do not move on a merge. `:latest` names the newest **promoted**
build and is repointed only by `promote.yml`, dispatched by hand from the Gitea Actions
UI with a version input. Promotion re-points `:latest` at the digest already running on
the canary — it never rebuilds — pushes a `promoted/vX.Y.Z` receipt tag, and creates a
GitHub release on the public `hyzr-dev/slusk` mirror. Rollback is the same workflow with
a lower version; it skips the release. See `docs/adr/0003-promote-by-digest.md`.

Two consequences worth carrying into every change:

- The release notes on a promotion are the **only** channel to the people running slusk.
  A change that adds a required config key stops their container from starting, and
  nothing else will warn them.
- Promoting is a judgement call about whether the canary looks healthy, and health here
  means `album_jobs` still moving — not that the process is up. A build can run for days
  and import nothing.

## The local PR lab is the substitute for staging

`testenv/` runs the full stack — this checkout's slusk, plus Lidarr, slskd and
Postgres — against real Soulseek searches, with no production data involved. Use it to
verify a PR before merging, since merging is the deploy.

```bash
cp testenv/.env.example testenv/.env   # first time: fill in two Soulseek test accounts
./testenv/lab.sh reset                 # clean run of the current checkout
./testenv/lab.sh info                  # addresses, accounts, listen ports
./testenv/lab.sh logs slusk
./testenv/lab.sh down                  # stop, keep state; `destroy` wipes volumes too
```

`reset` rebuilds from the working tree, wipes all state and seeds Lidarr with exactly
150 wanted albums from a fixed artist list, so two runs are comparable. `up` keeps state.

- **Two distinct Soulseek accounts are required.** Soulseek permits one login per
  account and both clients log in regardless of backend. Never use your own account.
- The lab defaults to `SLUSK_BACKEND=soulseek` (the native client), matching the value
  `config.example.toml` ships and the README recommends. `pipeline.backend` itself has
  **no default at all** — it is required (#396), so "the app's default" is not a thing
  to reason about; the example template's value is what a new user actually gets.
  Both backends carry a caveat, for different reasons, and neither should be described
  as the proven one: the native client is young and under active development, and
  slusk's slskd *adapter* (not slskd, a mature project) is a small, static piece of code
  on a fraction of the test coverage that the lab does not exercise by default.
- Results are not hermetic: peer availability and transfer speed vary between runs, so
  a `FAILED` job is evidence to investigate, not proof of a regression.
- Container logs echo the Soulseek usernames. Don't paste lab output verbatim into
  issues or PRs.
- `testenv/.env` and `testenv/runtime/` are gitignored and hold real credentials.

The observable surfaces are the dashboard on `:9090`, `/status`, two separate HTTP
endpoints that both carry job data, and the Postgres database.

- `/api/stream` is server-sent events (`internal/observ/stream.go`, issue #161): live

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyzr-dev/slusk](https://github.com/hyzr-dev/slusk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
