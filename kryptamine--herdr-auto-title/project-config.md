---
trigger: always_on
description: Herdr Auto Title — a Herdr plugin, written in Go, that generates tab titles from
---

# CLAUDE.md

Herdr Auto Title — a Herdr plugin, written in Go, that generates tab titles from
each tab's current context. Long-running process that polls the Herdr session,
no LLM and no external service.

## Language rule (mandatory)

**Everything written into this repository is in English.** Code comments, commit
messages, log and error messages, documentation, test names, ticket text — all
English, with no exceptions. This holds regardless of the language the request
was made in; only the conversation with the user follows the user's language.

## Commit convention (mandatory)

Commits follow [Conventional Commits](https://www.conventionalcommits.org):

```
<type>(<optional scope>): <subject>

<optional body explaining why, wrapped at 72 columns>
```

Types in use: `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `chore`.
Scope is a package or area (`resolver`, `state`, `herdr`, `app`).

- Subject in the imperative mood, lowercase, no trailing period, ≤72 characters
  ("add manual rename protection", not "Added manual rename protection.").
- The body explains why, not what — the diff already says what.
- One logical change per commit.
- Never add a co-author trailer.

## Type rule (mandatory)

**A struct field exists only if code reads it.** Herdr's wire objects carry far
more than Auto Title needs; mirroring them in full makes a type claim a
dependency the code does not have, and every unread field is a promise to keep
something working that nothing exercises. Add a field when the code that reads
it lands in the same change, and delete a field the moment its last reader goes.
The same holds for methods, constants and event payload types.

## Script rule (mandatory)

**Everything in `scripts/` is Python 3 and uses the standard library only.**
Shell stays where it belongs: the one-line recipes in the Makefile. Anything
with a loop, a branch or a data structure is a Python script.

Two scripting languages in one repository means two sets of portability traps to
remember — `stat -f` against `stat -c`, `trap` against signal handlers, quoting
rules that differ per shell — for tooling nobody should have to think about.
Python was already here for the probes, so it is what the rest is written in.

Each script is executable, opens with `#!/usr/bin/env python3` and a module
docstring saying what it is for, and takes no dependency outside the standard
library.

## Comment rule (mandatory)

**A comment is at most three lines.** That is a hard cap, in every language in
the repository. It is not a style preference: a comment long enough to need a
fourth line is explaining something the code cannot hold, and that explanation
belongs in [docs/architecture](docs/architecture/) where it can be read by
someone who is not already staring at the function.

Comment what is **surprising**, never what is visible:

- **Delete it** if it restates the code, names what a well-named identifier
  already names, or records where a value came from (which schema, which
  ticket, which measurement session). Provenance is what `git log` and
  `docs/architecture` are for.
- **Keep it** if a reader would otherwise "fix" the code and break it: a
  measured constant, a constraint the API imposes, an ordering that matters, a
  case that looks unhandled and is not.

When a decision genuinely needs a paragraph, write the paragraph in
`docs/architecture` and leave one line in the code pointing at it.

## Commands

```sh
make            # list every target
make check      # fmt + vet + lint + test   ← the gate before any commit
make lint       # golangci-lint, pinned in tools/go.mod
make test       # go test -race ./...
make run        # build and run in the current Herdr session, DEBUG logging
make dev        # the same, restarting on every source change
make ps         # show running plugin/watcher instances
make stop       # stop them
make tabs       # current tab names
make watch-tabs # ...refreshed every second
make probe-snapshot # the session snapshot the plugin polls
```

`go test -race` is the gate, not `go test`: the poll loop and the change history
it keeps are exercised concurrently in tests, and a future reset action will
touch that history from outside the loop.

The linter lives in `tools/go.mod`, a module of its own, so its dependency tree
stays out of the plugin's: the main module keeps two dependencies and still
builds on Go 1.24, which is what Herdr needs at install time. `errcheck` is off
— the places that swallow an error say why they do.

## Herdr socket API — verified facts

The originating specification is wrong on several protocol details. These were
verified against Herdr 0.8.2, protocol 20. **Probe before assuming anything not
listed here** (`make probe-*`, `scripts/probe.py`).

- NDJSON over the socket at `HERDR_SOCKET_PATH`. Requests are
  `{"id","method","params"}` — `params` is required even when empty.
- **One request per connection.** Herdr closes the connection after answering,
  so every `Call` dials its own. Auto Title uses three methods and no others:
  `session.snapshot`, `pane.process_info` and `tab.rename`.
- **The event stream is not used, on purpose.** `events.subscribe` replays a
  backlog before anything live — about the last 95 revisions of *every* pane at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kryptamine/herdr-auto-title](https://github.com/kryptamine/herdr-auto-title) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
