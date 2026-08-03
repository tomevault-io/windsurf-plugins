---
trigger: always_on
description: Guidance for AI agents (and new humans) working on this codebase.
---

# AGENTS.md

Guidance for AI agents (and new humans) working on this codebase.

## What awsmux is

awsmux runs one AWS CLI command across a whole fleet of accounts/regions,
safely. It discovers profiles from the AWS shared config and credentials
files, verifies every target identity with STS before anything runs,
classifies each operation by risk, gates anything non-read-only behind an
immutable plan + human approval token, fans out with a worker pool, and
persists every run to a replayable history. Agents consume it over MCP (`awsmux mcp`); humans use the CLI.

Single Go module (`github.com/0hardik1/awsmux`, so `go install` works
against the public repo), Go 1.26. Dependencies: **stdlib plus cobra,
nothing else** — this is a deliberate design rule, not an accident. Do not
add dependencies (the MCP layer is hand-rolled JSON-RPC on purpose, and the
AWS shared-config INI parsing is hand-rolled on purpose).

## Build, test, verify

```sh
go build ./... && go vet ./... && go test ./...
```

Run that before every commit, or use the make equivalents: `make build`
(binary at `./bin/awsmux`), `make test`, `make vet`, `make check-fmt`, and
`make lint` (golangci-lint, version pinned in the Makefile). `make setup`
installs the git hooks and pre-warms the lint tool. Tests live next to the
code (`internal/core/*_test.go`, `internal/mcpserver/*_test.go`); they are
plain stdlib `testing`, no test frameworks, and need neither network nor
Docker.

To try the whole thing end to end with zero credentials and zero real AWS
(needs Docker and the aws CLI):

```sh
make fleet-up                    # LocalStack + a 101-profile test fleet
source .tmp/fleet/env.sh && ./bin/awsmux targets
make e2e                         # build + fleet-up + smoke test
make fleet-down                  # remove the LocalStack container
```

## Layout

```
main.go                    thin entry: os.Exit(cmd.Execute())
cmd/                       cobra CLI layer, one file per command
  root.go                  root command, ExitError, shared selector/exec flags
  run.go plan.go approve.go apply.go   the plan/approve/apply workflow
  targets.go history.go replay.go     discovery + history + replay
  doctor.go                `awsmux doctor` environment diagnostic
  mcp.go                   `awsmux mcp` -> internal/mcpserver.Serve
  interactive.go           TTY checkbox target picker + typed confirmations
internal/core/             THE ENGINE - everything meaningful lives here
  types.go                 all shared types + stable exit codes
  discovery.go             config + credentials INI parsing/merge, glob selectors
  identity.go              STS preflight, 5m cache, dedup, re-verification
  classify.go              verb -> risk class tables + service overrides
  plan.go                  immutable plans, sha256 hash, ULID-style IDs
  policy.go                approval tokens, CheckApproval gate
  executor.go              worker pool, failure taxonomy, arg validation
  store.go                 ~/.awsmux persistence (plans/, executions/, index.jsonl)
  awscmd.go                aws CLI invocation: AWSMUX_AWS_BIN override,
                           then PATH, then well-known install locations
  doctor.go                environment diagnostic behind `awsmux doctor`
internal/mcpserver/        MCP stdio server: 5 tools, hand-rolled JSON-RPC 2.0
  server.go tools.go       framing + tool schemas/handlers
  results.go               token-economy result shaping (grouping, paging)
  registry.go              in-flight async execution registry
internal/output/           table | json | jsonl rendering (jsonl = agent/CI contract)
scripts/fleet/             LocalStack test-fleet provisioner (stdlib-only dev tool)
scripts/e2e.sh             smoke test run by `make e2e` and the CI e2e job
Makefile                   build/test/lint/fleet-up/e2e/hooks targets
.githooks/                 pre-commit (fmt, vet, lint) + commit-msg (Conventional Commits)
.github/                   CI workflow + dependabot
docs/ARCHITECTURE.md       design decisions, plan-boundary sequence, test-fleet internals
```

Layering rule (from `internal/core/types.go`): everything an agent or the
CLI can do goes through `internal/core`. `cmd/` and `internal/mcpserver/`
are thin wrappers — if you find yourself putting policy, classification, or
execution logic in either, it belongs in core instead, so both consumers see
identical behavior.

## The safety model — invariants you must not weaken

This project's entire value is that an AI agent holding admin credentials
cannot mutate anything without a human in the loop. When editing, preserve
these properties (and if a change touches one, say so explicitly in the PR):

1. **Classification fails safe.** `core.Classify` maps operation verbs to
   `read_only` / `mutating` / `destructive`; anything unrecognized is
   `unknown`, which policy treats as mutating. Service override tables
   (`stsClass`, `s3apiLocalWrite`, `s3Class`) exist where verb naming lies —
   e.g. `sts assume-role` and `s3api get-object` look read-ish but are
   classified mutating. Extend the tables; never loosen a default.
2. **`RequiresApproval`: only `read_only` runs freely.** There is no code
   path — no flag, no env var — that executes a non-read-only plan without a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0hardik1/awsmux](https://github.com/0hardik1/awsmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
