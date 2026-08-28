---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Start in a worktree

**Always begin work in a git worktree, never directly on a checkout of `main`.** Create one before
the first edit, do the whole change there, and open the PR from that branch.

```sh
git worktree add .claude/worktrees/<short-topic> -b <short-topic>
```

`.claude/worktrees/` is gitignored and is where existing worktrees live. In Claude Code, the
`EnterWorktree` tool does the same thing and switches the session into it.

## Stopping a dancer: gracefully, and never by pattern

dancer runs agents that work on *this repo*, so an agent's cleanup command can stop the
dancer that is running it. Two rules.

**Shut it down, do not kill it.** SIGTERM is the contract: dancer notifies live threads,
lets in-flight tool calls finish for `drain_timeout` (default 2m), persists final state and
exits 0 — and interrupted tasks then resume themselves on the next start. `kill -9` skips
all of that, cutting tool calls mid-write and leaving tasks that have to be picked up by
hand. Always wait for the process to actually exit instead of assuming it is gone:

```sh
kill "$pid"                                   # SIGTERM: drain, persist, exit 0
while kill -0 "$pid" 2>/dev/null; do sleep 1; done
```

For the deployed service, let systemd do it — it sends SIGTERM and waits `TimeoutStopSec=150`:

```sh
sudo systemctl stop dancer        # or: restart
```

**Never find a dancer by command-line pattern.** `-f` matches anywhere in the command line, so
`"bin/dancer run"` also matches the deployed `/usr/local/bin/dancer run`. This has taken the
production instance down mid-task twice — the second time via `pgrep -f "bin/dancer run"`
followed by `kill <pid>`, so killing "by pid" is no safer when the pid came from a pattern.
`pgrep`, `pkill`, `ps | grep` and `killall` are all the same hazard.

Keep the pid from the process you started, and use only that:

```sh
env DANCER_CONFIG=/tmp/dancer-test/config.toml bin/dancer run & pid=$!
# ... test ...
kill "$pid"; while kill -0 "$pid" 2>/dev/null; do sleep 1; done
```

If you truly have no pid, anchor to the absolute path you launched and check what you matched
before signalling anything:

```sh
pgrep -af "^/tmp/dancer-test/bin/dancer run"   # -a: read it first, confirm no /usr/local/bin
```

A `pgrep`/`pkill` pattern that could match `/usr/local/bin/dancer` is always a bug.

## Commands

```sh
make build            # bin/dancer
make test             # go test ./...
make test-race        # go test -race -count=1 ./...
make lint             # gofmt -l check + go vet   (run before finishing a change)
make fmt tidy
make run              # bin/dancer run -config $CONFIG   (Slack)
make run-terminal     # same, but the terminal transport — the fastest way to try a chat change
make run-web          # same, but the web transport (browser UI on web.listen)
make doctor           # config, claude login, docker, ssh hosts, Slack tokens
make help             # every target
```

Single test / package:

```sh
go test ./internal/coordinator -run TestName -v
go test -race -count=1 ./internal/coordinator     # coordinator is concurrent; use -race
DANCER_LIVE=1 go test -count=1 ./internal/agent/claude   # drives the real claude CLI (~cents, haiku)
DANCER_LIVE=1 go test -count=1 ./internal/decider ./internal/coordinator -run Live   # real decider verdicts
make e2e              # scripts/e2e.py: whole binary through the terminal transport
make restart-drill    # scripts/restart-drill.py: SIGTERM mid-tool-call → drain → resume
```

`DANCER_DOCKER_PROVISION=1 go test ./internal/environment/docker` builds a real image from
`ubuntu:24.04` (~60s, downloads packages); it is skipped otherwise.

Tests that need real infrastructure skip themselves rather than fail: docker tests need a live
daemon, ssh tests spin up a throwaway `sshd`, live claude tests need `DANCER_LIVE=1` plus a
logged-in `claude`. Keep that pattern for new integration tests.

Config path is `$DANCER_CONFIG`, else `~/.config/dancer/config.toml`. `deploy/config.example.toml`
is the reference for every key.

## Architecture

One Go binary (`cmd/dancer`: `run` | `setup` | `doctor` | `user`). Data flows in one loop:

```
transports --Inbound--> surfaces --Intent--> Coordinator --Task--> Executor --> Agent --> Environment
transports <-Outbound-- surfaces <--Event--- Coordinator <-----agent.Event------------------┘
                                                 ↓
                                            Store (SQLite event log + projections)
```

Each layer is an interface defined in the package doc of `internal/<pkg>/<pkg>.go`; read those
files first — they carry the contract, the concrete packages under them are implementations.

- **`transport`** (slack, web, terminal) — dumb on purpose: text, prompt-with-choices, files, `ThreadID`
  (Slack: `"<channel>/<thread_ts>"`; web: `"<channel>/<id>"`). It never interprets a message.
  Files go both ways: `Outbound.Files` are uploaded after the text; `Inbound.Files` are the
  attachments a human sent, downloaded by the transport (Slack: `files:read`), and the executor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cleanunicorn/dancer](https://github.com/cleanunicorn/dancer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
