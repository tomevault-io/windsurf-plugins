---
trigger: always_on
description: `chromedp` is a Go package that drives Chrome and other browsers that speak
---

# chromedp

`chromedp` is a Go package that drives Chrome and other browsers that speak
the Chrome DevTools Protocol. The protocol is the JSON message format that
Chrome uses for remote control. `chromedp` needs no external driver. It
starts the browser or connects to one, then sends protocol commands over a
pipe or a WebSocket.

The generated protocol types live in a separate module, `cdproto`, which
`pdlgen` writes. The core module of this repository holds the high level API on
top of it: allocators, contexts, actions, selectors, input and screenshots. The
core uses only the Go standard library and `cdproto`. See
`docs/decisions/2026-10-04-the-core-uses-only-the-standard-library.md`.

The repository holds three modules:

| Directory | Module | Holds |
| --- | --- | --- |
| the root | `github.com/chromedp/chromedp` | the core, with the pipe transport |
| `remote/` | `github.com/chromedp/chromedp/remote` | everything that needs a websocket, and the library `gobwas/ws` |
| `test/` | `github.com/chromedp/chromedp/test` | the tests that need `pdf` or `pixelmatch` |

## Standing rules

These hold in every `chromedp` repository, for every coding agent.

1. Stage changes for review. Commit and push only when the maintainer says so.
2. Load the `simple-english` skill before you write text that a person reads.
   Examples are a document, a code comment, an error message and a commit
   message. Follow the skill for that text.
3. Load the `go-pedantry` skill before you write or review Go code. Follow it
   where it does not conflict with a rule in this file. A rule here wins.

Questions and feature ideas go to GitHub Discussions, and bugs go to issues. Do
not open an issue for a question. See
`docs/decisions/2026-10-03-questions-and-ideas-go-to-discussions.md`.

`CLAUDE.md` holds one line that imports this file, so that Claude Code and
every other agent read the same rules. Edit this file, not that one.

## Which document to read

Read `docs/PLAN.md` first. It describes the purpose and the architecture. Its
last section lists the open questions. Do not decide an open question on your
own. Ask the maintainer.

`docs/decisions/README.md` is the index of every decision. A decision is one
file named by its date and a short title. Read the status of a decision before
you trust it, because a later decision can amend or replace it.

| If you are | Read |
| --- | --- |
| starting work | `docs/PLAN.md`, then `docs/PROGRESS.md` |
| looking for known work that is not done | `docs/BACKLOG.md` |
| asking why something is the way it is | the index in `docs/decisions/README.md` |
| changing the public API | `docs/PLAN.md` under Architecture, `docs/API.md`, `docs/MIGRATION.md`, then the decisions in `docs/decisions/` |
| looking for the old and the new name of an API | `docs/MIGRATION.md` |
| looking for the API with before and after code | `docs/API.md` |
| changing the cdproto dependency | `docs/decisions/2026-10-03-stop-relying-on-removed-cdproto-helpers.md` |
| changing the README | `docs/decisions/2026-10-03-readme-shows-the-gear-logo-and-discord-badge.md` |
| recording a decision | `docs/decisions/README.md` and `docs/decisions/2026-10-03-use-dated-decision-files.md` |
| running the tests | the section Before you commit, in this file |

## Hard rules

1. Do not run the tests in a session that has no browser. They need Chrome or
   the `chromedp/headless-shell` container. A session without one must run
   `go build ./...`, `go vet ./...` and `go test ./docs/` only.
2. Do not edit `kb/kb.go` or `device/device.go` by hand. `go generate` writes
   them from `kb/gen.go` and `device/gen.go`. Change the generator and run it.
3. Hold the lock before you read a field of a `chromedp.Node` or a
   `chromedp.Frame`. The code in `query.go` and `target.go` calls `RLock`
   first, because the target updates the node tree from events while an action
   reads it.
4. Run an action with `chromedp.Run` or `chromedp.Do`. They start the browser
   and the tab when the context has none, and they pass the `Target` to the
   action. Do not call an action with a `Target` that these funcs did not
   prepare.
5. Wrap every error with `%w`. See the decision
   `docs/decisions/2021-04-29-wrap-errors-with-w.md`.
6. Keep the tests compatible with `headless-shell`. CI runs them against Chrome on
   Linux, Windows and macOS, and against the `chromedp/headless-shell` image on
   Linux. A test that fails on one
   of them must skip with a comment that says why. `input_test.go` shows how.
7. Embed JavaScript from the `js/` folder with `go:embed`. Do not write
   JavaScript inside a Go string.
8. Do not change the type `Action[T]` or the signature of an exported
   function without asking the maintainer. See `docs/decisions/2026-10-03-generic-iterator-api-instead-of-action.md`.
9. Do not add a helper that the `cdproto` module is dropping. See
   `docs/decisions/2026-10-03-stop-relying-on-removed-cdproto-helpers.md`.

## Layout

The root package `chromedp` holds the API. The files group by topic.

| Path | Holds |
| --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chromedp/chromedp](https://github.com/chromedp/chromedp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
