---
trigger: always_on
description: > Agent operating context for this repo (agents.md spec). Complements `README.md`
---

# AGENTS.md — gitmoot

> Agent operating context for this repo (agents.md spec). Complements `README.md`
> (which is for humans). `CLAUDE.md` imports this file via `@AGENTS.md`.
>
> This is the filled-in **Project Map** for the `lead-engineer` work strategy.
> Accuracy is sacred: every claim here is verified against the repo or describes
> this host's live deployment. Don't add a claim you haven't checked.

## Project overview

gitmoot is a local-first coordinator for AI coding agents working across GitHub
repositories, pull requests, goals, reviews, and runtime workflows. It ships as a
single static Go binary plus a background daemon; workflow state lives in local
SQLite (the **modernc pure-Go** driver — no cgo). The single static binary with
**zero runtime dependencies** is a core invariant.

It drives four runtimes (`codex`, `claude`, `kimi`, `shell`). `agent start`
supports codex/claude/kimi; `shell` is a subscribe-only command runtime used
mainly to drive engine-feature E2Es with no LLM.

## Build, test, and verify (the gate)

Requires Go 1.26+ (see `go.mod`; CI resolves the version via
`go-version-file: go.mod`). On this host, pin the toolchain:

```sh
export GOTOOLCHAIN=local PATH=/root/.local/toolchains/go1.26.4/bin:$PATH
export GOCACHE=/tmp/gitmoot-go-build-cache
mkdir -p "$GOCACHE"
```

Run from the repo root and make these pass before committing — they mirror the CI
gate in `.github/workflows/ci.yml`:

```sh
go build -buildvcs=false ./...
go generate ./... && git diff --exit-code   # gitmoot_result contract is single-sourced + regenerated; stale artifact fails CI
go vet ./...
go test -timeout 25m ./...

# Race gate is scoped (not ./...). Use CI's package shard counts and its
# deterministic fallback partitioner so no growing package hits one monolithic
# timeout. Each compiled binary covers every package test exactly once.
(
  set -e
  race_dir="$(mktemp -d "${TMPDIR:-/tmp}/gitmoot-race.XXXXXX")"
  printf 'race artifacts: %s\n' "$race_dir"
  for spec in cli:8 pipeline:4 db:2 workflow:4 daemon:1; do
    package="${spec%%:*}"
    shards="${spec##*:}"
    bundle="$race_dir/$package"
    mkdir -p "$bundle/partitions"
    go test -c -race -o "$bundle/$package.test" "./internal/$package/"
    (
      cd "internal/$package"
      "$bundle/$package.test" -test.list '.*'
    ) >"$bundle/tests.list"
    scripts/partition-race-tests.sh \
      --tests "$bundle/tests.list" \
      --shards "$shards" \
      --out-dir "$bundle/partitions"
    for ((shard = 0; shard < shards; shard++)); do
      run_regex="$(cat "$bundle/partitions/shard-$shard.regex")"
      (
        cd "internal/$package"
        "$bundle/$package.test" \
          -test.run="$run_regex" \
          -test.timeout=20m
      )
    done
  done
)
```

Managed-worktree runtime seats append `-buildvcs=false` to inherited `GOFLAGS`
so stray ancestor `.git` directories cannot confuse Go's VCS root detection.

The explicit temporary `GOCACHE` is also part of the host setup. Managed
worktrees can inherit a read-only `/root/.cache/go-build`; redirecting the cache
keeps build, vet, and test from failing during package setup before compilation.
This is documented instead of changing `/root/.cache` permissions because that
directory is host-global external state, while the gate must remain runnable in
each agent's environment.

The race block deliberately does not delete `race_dir`: recursive deletion is
policy-rejected for managed coordinators, and cleanup is not required for gate
correctness. The printed per-run directory and
`/tmp/gitmoot-go-build-cache` therefore persist for later owner-managed cleanup.

When the repository checkout itself is under `/tmp`,
`TestClaudeProduceHookAutoReadLandlockE2E` is a known host-environment confound:
it fails there on current `main` as well as feature branches, so that failure is
not evidence about the branch under test. In a `/tmp` checkout, run the non-race
gate with that one test explicitly skipped:

```sh
go test -timeout 25m -skip 'TestClaudeProduceHookAutoReadLandlockE2E' ./...
```

`-buildvcs=false` is required, not optional, inside a gitmoot worktree (#1209):
Go's VCS auto-stamp only recognizes a `.git` **directory** as a repo root
(`cmd/go/internal/vcs.vcsGit.RootNames`), but a linked worktree's `.git` is a
**file** (a `gitdir:` pointer). Go's root-detection walk-up skips past the
worktree's real root looking for any ancestor with a `.git`-shaped directory,
and can land on an unrelated one — hard-failing with `error obtaining VCS
status: exit status 128` even though `git status` itself works fine from the
same directory. This is a genuine Go toolchain limitation with linked
worktrees (confirmed by reading `cmd/go/internal/vcs/vcs.go`'s `FromDir` /
`isVCSRoot`), not something gitmoot's code or config can fix, and even the
non-failing cases can silently stamp the wrong VCS metadata (wrong commit,
wrong dirty bit) from whatever directory the walk-up happened to land on.
Disabling it here costs nothing real: release binaries get their version
info from the explicit `-ldflags -X ...Commit=$(git rev-parse HEAD)` recipe
in the deploy section below, never from Go's auto-stamp.

`-timeout 25m` on the plain `go test ./...` closes the same kind of gap

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitmoot/gitmoot](https://github.com/gitmoot/gitmoot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
