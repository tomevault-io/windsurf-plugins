---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

Trau is an autonomous, ticket-driven development loop shipped as a single static Go binary (`CGO_ENABLED=0`). It picks a ready tracker ticket (Linear/Jira/GitHub Issues, or an internal issue when no external tracker is configured), drives it through **build → handoff → verify → commit → PR → CI → merge** with one fresh AI-agent subprocess per phase, checkpoints after each phase, and resumes after any interruption.

Read `CONTEXT.md` first — it defines the domain vocabulary (**Provider / Model / Route / Fallback provider / Provider override**) and which near-synonyms to avoid. Use those terms exactly in code, comments, and tickets.

## Language

- **Write in ASD-STE100 Simplified Technical English.** This applies to everything you write: code, comments, commit messages, PR bodies, tickets, and replies to the operator. Keep one idea per sentence, prefer the active voice, use an approved word only in its approved part of speech, and never reach for a synonym of a term you have already used.
- **Read every `CONTEXT.md` before you write, and use its ubiquitous language.** The terms it defines beat any near-synonym, in prose and in identifiers alike. A term the domain model does not define is a term you must not invent on the spot.

## Commands

```bash
make build                 # compile to bin/trau
make reset                 # rebuild + `trau hub restart` — humans only; refuses inside a trau run (TRAU_ACTIVE=1)
make test                  # go test -race ./... — the pre-PR gate; run it locally, nothing runs it for you (TESTFLAGS='-timeout 8m'; see the Tests bullet)
go test -race ./internal/pipeline -run TestVerifyPause   # single test
make vet                   # go vet ./... (runs the make windows gate first)
make windows               # GOOS=windows go build ./... — native-Windows compile gate (ADR 0023)
make lint                  # golangci-lint (installed separately)
make fmt                   # gofmt -w .
make dist                  # cross-compile release matrix into dist/
```

**Rebuild after every feature or ticket** so the change is actually testable. `make build` refreshes `bin/trau`. A running `trau serve` is a port-locked singleton with the SPA compiled into the binary, so it keeps serving the old code until restarted — autostart *reuses* a stale hub, it does not replace it. The web UI's footer shows `web`/`cli` versions; after a rebuild, confirm they reflect the new build (hard-refresh the browser, since the bundle's asset names are not content-hashed).

**The web bundle is generated, never committed.** `internal/webserver/dist` is git-ignored. A tracked bundle made every two deliveries that touch `web/` collide on a binary file that no merge can resolve, which is why it is out of the index now. A fresh clone needs one step: `make build`. It runs the SPA build (`npm ci && npm run build`) and then links the binary, so the hub it produces serves the web UI. `make test` builds the SPA first for the same reason. Only `internal/webserver/dist/.gitkeep` stays tracked, because `//go:embed all:dist` must match one file to compile; the SPA build writes the placeholder back, so a build no longer dirties the working tree. A binary from a bare `go build ./...` still compiles and runs — it answers each page route with a 503 that names `make build`. The release path does not change: `.goreleaser.yaml` rebuilds the bundle in its `before` hooks.

**The hub on :8728 is off-limits during agent runs.** If you are an agent spawned by trau itself (a build/verify/repair/cleanup phase — `TRAU_ACTIVE=1` is in your env), the hub on :8728 is the orchestrator that owns your run: restarting or killing it destroys the run's data channel and pauses the whole loop. The only supported restart is `trau hub restart`, which asks the running hub to re-exec the on-disk binary — never `kill`, never a port grab. `make reset` is `make build` plus that command and refuses outright while `TRAU_ACTIVE=1`; `trau hub restart --force` (the escape hatch for a hub whose API has wedged) refuses in the same situation, and while any loop is live. Both refusals are by design — do not work around them. To QA hub/web changes live, start an **isolated** hub from the dev binary and drive only that instance:

```bash
iso=$(mktemp -d) && TRAU_HOME=$iso/.trau HOME=$iso ./bin/trau serve --port 8799
```

Override BOTH `TRAU_HOME` and `HOME` so the isolated hub cannot see the real `~/.trau` databases. When done, kill only that pid. Never stop, restart, or "clean up" any process listening on :8728 — it was not started by you, even if a port probe makes it look recent. `make reset` is for humans doing interactive dev outside a run.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RomkaLTU/trau](https://github.com/RomkaLTU/trau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
