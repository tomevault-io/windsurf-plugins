---
trigger: always_on
description: Issues live in the MAC hub task ledger (`mac task`), which is the
---

# Agent Instructions

Issues live in the MAC hub task ledger (`mac task`), which is the
canonical execution store. `.tickets/` is ignored local operational
state for migration/compatibility workflows only; do not rely on it as
a checked-in source of truth and do not create or commit `.tickets/`
files during normal work.

This project does **not** use beads (`bd`) or dolt — issue tracking is the
**mac task ledger** (`mac task`), a beads-equivalent durable ledger that avoids
the beads/dolt sync problems. It is correct to say we do not use beads; it is
**not** correct to say we lack a durable task ledger — we use mac tasks. The
legacy beads read/write bridge is not the normal execution path, dolt sync is
disabled, and `.beads/` has been removed. Remaining beads commands are for
read-only detection and one-way migration. Do not run `bd`; use `mac task`.

## Fleet Host Resolution

`~/.mac/fleets.yaml` is the definitive source of truth for fleet agent targets.
When checking, refreshing, deploying to, or SSHing into a fleet agent, resolve
the agent's current `target` from that file first. Do not assume a hostname from
the agent name, local SSH aliases, known_hosts entries, old fleet backups, hub
history, or prior conversation context; those can be stale after host swaps.

## Quick Reference

```bash
# Read issues
mac task ready --limit 10                    # open + deps done + unclaimed + dispatchable
mac task stats                               # counts by state
mac task search <keyword>                    # title/description match

# Projects
mac project create <project> --active        # manual project, immediately dispatchable
mac project onboard <repo-url> --project=<project>  # creates contract-authoring task
mac bridge repository register <name> <path> --project=<project>
mac bridge repository repos
mac project activate <project>               # clear project-level dispatch pause

# Lifecycle
mac task create "title" --project=<project> --description-file=desc.txt --metadata-file=meta.json
mac task create "title" --no-dispatch        # stage task; writes metadata.no_dispatch=true
mac task release <task_id>                   # clear no_dispatch so fleet can claim it
mac task break-glass <task_id> <agent_id> --reason="..."  # admin, single-use direct-host recovery
mac task break-glass-list <task_id>          # inspect durable recovery authorization
mac task break-glass-revoke <auth_id> --reason="..."
mac task claim <task_id> <agent_id>
mac task start <task_id> <agent_id>
mac task show <task_id>                      # detail + history
mac task close <task_id> --reason="..."
mac dispatch tick --limit 10                 # ask dispatcher to assign ready work now

# Memories (cross-session knowledge)
mac memory remember <key> "<content>" --project=mac
mac memory list --project=mac
mac memory forget <key> --project=mac

# Inspect or migrate other repos that still have legacy beads state
mac task detect-beads <repo>
mac task migrate-beads <repo> --project=<name> --tickets-only
```

## Fleet Operational Learning

Treat operational outcomes as shared control inputs, not disposable log lines.
Repository-access attempts record secret-free `mac.fleet_learning.v1` memories;
reviewer routing prefers recent success and temporarily avoids a newer
authentication failure. Do not repeatedly retry the same credential pattern on
the same agent. Allow the workflow to choose a peer with a proven success, or
repair the credential and supersede the failure with a successful attempt.

Never store credential values, authenticated URLs, or raw secret-bearing
command output in memory. Store only the credential source name, redacted host,
operation, outcome, classified failure, and actionable remediation. See
`docs/fleet-operational-learning.md` for the schema and lifecycle contract.

> Pass multi-line / shell-hostile content (parens, backticks, `$VAR`, newlines) via the `--<name>-file` variants instead of inline quotes. `--<name>-file -` reads from stdin.

`no_dispatch` is a hold flag, not a lifecycle state. The held form is
`metadata.no_dispatch=true`; `mac task release` removes that key instead of
writing `false`. A task with no `no_dispatch` key is dispatchable, subject to
dependencies, worker capabilities, leases, and project dispatch pause.

To tell agents to work on a project, create or onboard the project, create
project-scoped tasks, make sure the project is active, release any staged
tasks, and let loop-mode agents claim from `mac task ready`. Use
`mac dispatch tick` for an immediate dispatcher pass, or `mac task claim` /
`mac task start` when assigning a specific agent manually.

## CodeGraph Runtime Baseline

CodeGraph is a legitimate runtime assumption in the default agent environment.
Fleet deploy installs `codegraph`, runs `codegraph install`, and fails the
deploy if CodeGraph cannot be prepared; the OpenShell agent image does the same
at image build time. Agents may use CodeGraph to understand repository APIs,
code behavior, call relationships, and skills that benefit from code structure.
Use it as analysis support, not as a replacement for reading source files and
running tests.

When analyzing a repository, run `codegraph init` if the index is absent or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanhubbard/mac](https://github.com/jordanhubbard/mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
