---
trigger: always_on
description: Conventions for working in this repo (humans and agents).
---

# CLAUDE.md — muster

Conventions for working in this repo (humans and agents).

## What muster is

A local multi-agent coordination bus: independent coding-agent sessions (Claude
Code + OpenAI Codex, each in its own tmux tab) hand tasks/messages to each other —
no copy/paste, subscription-only (the bus never calls a model; it routes between
agents already running on their own plans). One static Go binary, multi-mode
(`serve` daemon · `mcp` stdio server · human CLI · `lambda` handler for the
optional hosted backend).

**License.** muster is BUSL-1.1 as of v0.18.0 — free for organizations under 25
employees, and each release converts to Apache-2.0 three years after it ships.
Versions before v0.18.0 stay MIT in perpetuity.

## Build / test / run

- **`just verify`** — the gate: `gofmt`, `golangci-lint`, `go test -race`, build,
  `cross` (all four release targets plus the `-tags lambda` build). Run it before
  every commit; CI runs the same recipe, so local and CI can't drift.
- **`just verify-dynamo`** — the second gate, deliberately NOT part of `verify`
  because it needs Docker. Runs `internal/dynamostore` and the DynamoDB half of
  the cross-backend conformance suite against DynamoDB Local. Without an endpoint
  those tests *skip*, so `verify` compiles and vets them but proves nothing about
  DynamoDB semantics — run this after touching `internal/dynamostore`. The
  `dynamo` job in CI runs the same two packages against a service container.
- **cgo-free** — the binary builds under `CGO_ENABLED=0` (pure-Go SQLite via
  `modernc.org/sqlite`). Don't add cgo dependencies.
- **macOS tests** use `internal/mustertest.ShortHome()` for unix-socket paths (the
  `sun_path` ~104-char limit; `t.TempDir()` is too long and breaks the socket).
- Build + run: `go build -o ~/.local/bin/muster ./cmd/muster`, then
  `muster serve | mcp | agents | send | inbox | tasks | nudge | register | status | update | commands | standing | channel | station | …`.

## Branch model

`feat/* → dev → main`. CI (`just verify`) is required on `dev` and `main`; `main` is
the release line. Never develop on `main` — do feature work in a git worktree off
`dev`, merged via PR.

**Releases are automated.** The `VERSION` file is the knob: bump it on `dev`, and
when the promotion PR merges to `main`, the release workflow tags `v<VERSION>`,
creates the GitHub release with generated notes, and attaches cross-compiled
binaries (darwin/linux × arm64/amd64) with checksums. A merge to `main` that
doesn't bump `VERSION` releases nothing. The darwin binaries are signed and
notarized by the release job itself, which is why that job runs on a macOS
runner, and it fails rather than publishing anything unsigned — so a green
release needs no manual signing step. `contrib/release-sign.sh v<VERSION>`
remains only for repairing an already-published release by hand.

Pick the next version from the TAGS, not from `dev`'s `VERSION` file. The
release commit bumps `VERSION` on `main` only, so until someone back-merges,
`dev` still shows the version before the last release and a feature branch cut
from it will propose a number that is already tagged — v0.14.0 was chosen this
way after v0.13.0 had shipped. CI's `version-guard` catches the collision, but
only once the PR is open.

## Architecture (the mental model)

- **daemon = API.** A lazy unix-socket daemon speaking newline-delimited JSON
  (`internal/proto`). The MCP server (`internal/mcpserver`) and the human CLI
  (`internal/cli`) are **peer clients** of the daemon — neither goes through the
  other. Any daemon op is reachable from a plain CLI subcommand.
- **tmux = substrate.** Liveness, wake, and identity lean on tmux — but only through
  `internal/tmuxenv` (the one canonical capture path) or the injected
  `wake.Notifier`. Keep `internal/daemon` and `internal/store` tmux-agnostic.
- **Wake is split.** `internal/wake` *notifies* (sets the `@muster_inbox` tmux
  option; never types into a pane). `internal/nudge` is the **only** send-keys path.
  The daemon never types. `internal/channel` is the third wake path: it pushes an
  envelope into the session over MCP (`muster channel`) — like `wake`, it never types.

## The naming contract

The tmux option pair `@claude_task` / `@claude_task_manual` is the neutral
meeting point between muster and the operator's dotfiles (spec:
docs/superpowers/specs/2026-08-05-conversation-identity-naming-design.md).
Intentional gestures (prefix T, `muster label`, the SessionStart projection
of a transcript custom-title) set both; automatic syncs write only the label
and defer to the flag; readers trust the pair. The conversation's transcript
custom-title is the one durable name — tmux and the bus are projections.
Attribution requires proof: `session_created = 0` never matches a live
session (tmuxenv.IsSessionAlive), while DepartStaleSiblings still spares
those rows from reaping — attribution and tombstoning are distinct decisions.

Naming and identity are separate questions. The naming contract above governs what a conversation is *called*; underneath it, a conversation's identity resolves by transcript path first, falling back to the live tmux pane tuple only when no row carries that transcript path yet.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schuettc/muster](https://github.com/schuettc/muster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
