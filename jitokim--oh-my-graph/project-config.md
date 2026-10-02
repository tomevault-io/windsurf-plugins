---
trigger: always_on
description: This is the project-level Claude Code config for **oh-my-graph** itself
---

# oh-my-graph — instructions for Claude Code

This is the project-level Claude Code config for **oh-my-graph** itself
(public, committed — no personal or private settings live here).

## What this project is

oh-my-graph is a graph-native multi-agent orchestrator: it runs each node of
a YAML-defined DAG as a real `claude -p` subprocess on the user's own
logged-in Claude subscription (never the metered Anthropic API). Nodes run
with session persistence on, so every node is also an ordinary claude session
in `~/.claude/projects` that any external tool can read.

Since ADR 0025 a run may instead select `--runtime codex`, which spawns
`codex exec` on the user's own Codex login by the same rule — one runtime for
the whole run, the provider's own CLI, that provider's saved login. Claude is
the default and the path every unqualified statement in this file describes.

**DESIGN.md is the spec.** Read it before touching the scheduler, the graph
schema, the `NodeRunner` interface, or handoff. If code and DESIGN.md
disagree, treat that as a bug in one of them, and fix both together — don't
let them drift apart.

## Load-bearing invariants — do not weaken these

- **Subscription-auth env scrub.** Every child process's environment must have
  `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `OPENAI_API_KEY` and
  `CODEX_API_KEY` deleted, via the shared
  `internal/childenv.Scrub` — used by ALL spawners (a model node, a
  `success_check.verify` command, the git commands behind a node's
  `worktree:`, and the browser launch of the `serve` URL). This is what keeps
  the tool on subscription billing instead of silently falling back to
  metered API billing. It is unit-tested in
  `internal/childenv/childenv_test.go` and at each call site
  (`internal/runner/claude_test.go`, `internal/verify/shell_test.go`,
  `internal/worktree/git_test.go`, `internal/browser/exec_test.go`) — don't
  touch env construction without keeping those tests meaningful.
  There is ONE list and NO runtime branch: a Claude node drops the OpenAI
  switches and a Codex node drops the Anthropic ones. Splitting it in two —
  "each runtime only needs its own" — is the tempting optimisation, and the
  half that then falls behind a new provider variable bills silently, with
  nothing failing to say so.
- **The four exec seams.** Exactly four objects may spawn a process:
  `runner.CLIRunner` (a node's model CLI subprocess, claude or codex),
  `verify.ShellVerifier` (a node's evidence command),
  `worktree.GitManager` (the `git worktree` commands behind a node's
  `worktree:`) and `browser.ExecOpener` (the `open`/`xdg-open` launch of the
  `serve` URL) — see `docs/adr/0002-verification-is-a-second-exec-seam.md`,
  `docs/adr/0005-worktree-provisioning-is-a-third-exec-seam.md` and
  `docs/adr/0006-browser-open-is-a-fourth-exec-seam.md`.
  Everything else (scheduler, CLI, handoff, ledger, coordinator) depends on
  the `NodeRunner`, `Verifier`, `worktree.Provider` and `browser.Opener`
  interfaces only, so the whole engine is testable via the scripted
  `FakeRunner`/`FakeVerifier`/`FakeManager`/`FakeOpener` with zero real
  spawns. A fifth spawner needs its own ADR.
- **Artifact handoff is the default.** `handoff: artifact` persists a node's
  result to `~/.oh-my-graph/runs/<run-id>/<node-id>.out` (base overridable via
  `OMG_HOME`); `handoff: session`
  (resuming `--resume <session_id>`) is opt-in and only valid with exactly one
  session-parent.
- **Never a provider SDK. Never `--bare`. Never `--no-session-persistence`.**
  A node runs as the provider's own CLI subprocess — `claude` or `codex`, one
  runtime for the whole run (ADR 0025) — on whatever login that CLI has saved,
  with session persistence on (so every node stays observable as an ordinary
  session transcript). A second runtime widens which CLI, never the rule: no
  direct model API, and no flag that detaches a node from its session.
  Be exact about what the billing guarantee IS: oh-my-graph guarantees the env
  scrub above and never passing `--bare`, so nothing IT does switches a CLI off
  its saved login. It cannot guarantee that login is OAuth — `codex login
  --api-key` writes a key into `~/.codex/auth.json`, which no scrub touches, and
  the same holds for any credential a CLI persists on disk. SECURITY.md and
  `internal/childenv/childenv.go` state it in that narrower form ("can make the
  selected CLI ignore its saved login"); this file must not state it in a wider
  one.

See [SECURITY.md](SECURITY.md) for the full ToS/security stance and
[CONTRIBUTING.md](CONTRIBUTING.md) for how these invariants are enforced in
review.

## Build / test / smoke

```sh
make build      # go build -o bin/oh-my-graph ./cmd/oh-my-graph
make test       # go test ./... -race -count=1 — FakeRunner only, no real claude
make vet        # go vet ./...
make fmt-check  # gofmt -l . (CI gate)
make smoke      # MANUAL ONLY: real claude, a few cents, never in CI
```

All engine logic (scheduler, DAG validation, handoff, retry, halt-on-fail) is
covered by `internal/runner.FakeRunner` fixtures. If you add or change
scheduling/runtime behavior, write the test against `FakeRunner`, not a real
`claude` spawn.

## Repo layout (see DESIGN.md "Repo layout" for the authoritative version)

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jitokim/oh-my-graph](https://github.com/jitokim/oh-my-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
