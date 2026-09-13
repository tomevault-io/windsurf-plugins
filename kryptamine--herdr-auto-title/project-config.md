---
trigger: always_on
description: Herdr Auto Title — a Herdr plugin, written in Go, that generates tab titles from
---

# AGENTS.md

Herdr Auto Title — a Herdr plugin, written in Go, that generates tab titles from
each tab's current context. Long-running process that polls the Herdr session,
no LLM and no external service.

## Repository layout

```
cmd/herdr-auto-title  the binary
internal/app          the poll loop and the reads it spends, configuration
internal/herdr        the socket client; herdrtest beside it is its stub
internal/state        a session snapshot turned into what each tab is doing
internal/resolver     that state turned into a title, one source at a time
internal/claude       what a Claude Code session is about, from its transcript
internal/git          what a repository has checked out, read from .git
scripts/              the Python probes
docs/architecture/    how the plugin works and why
```

Each package's doc comment says the rest.

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

## Branches and pull requests (mandatory)

**Never commit to `main`.** Branch from it first, named `<type>/<kebab-summary>`
with the types the commits use: `feat/optional-agent-name`,
`docs/security-policy`, `chore/tighten-the-linter-set`.

- **Pull requests are merged by rebase.** Merge commits and squashing are both
  disabled, and each broke something: GitHub puts the conventional PR title into
  a merge commit, so release-please counted every change twice, and a squash
  collapses a pull request into one changelog line, losing the granularity that
  "one logical change per commit" exists to produce.
- **`CHANGELOG.md`, the tags and the version in `herdr-plugin.toml` belong to
  release-please.** Never edit one by hand.
- Keep a pull request to one thing. A refactor, a feature and a formatting sweep
  are three pull requests.

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

**A comment is at most three lines**, in every language in the repository, and
it says what is surprising rather than what is visible. A decision that needs a
paragraph goes in [docs/architecture](docs/architecture/), with one line in the
code pointing at it.

The rule in full, in Go terms and with worked examples of what to keep and what
to delete, is in [.claude/rules/comments.md](.claude/rules/comments.md).

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

`go test -race` is the gate, not `go test`: the state a poll carries between
polls is shared, two tests still run the loop in a goroutine of its own, and a
future reset action will touch that state from outside the loop.

The linter lives in `tools/go.mod`, a module of its own, so its dependency tree

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kryptamine/herdr-auto-title](https://github.com/kryptamine/herdr-auto-title) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
