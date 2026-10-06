---
trigger: always_on
description: This repository holds 43 example programs for `chromedp`, a Go package that
---

# chromedp examples

This repository holds 43 example programs for `chromedp`, a Go package that
drives Chrome through the Chrome DevTools Protocol. The programs are larger
than the examples in the package documentation. They show how to solve a task
with `chromedp`: click an element, download a file, emulate a device, use a
proxy and more. The module is `github.com/chromedp/examples`.

The programs use the typed API of `chromedp` v0.20.0 and `cdproto` v0.157.8. See
`docs/decisions/2026-10-03-the-programs-use-the-new-typed-api.md`.

## Standing rules

These hold in every `chromedp` repository, for every coding agent.

1. Stage changes for review. Commit and push only when the maintainer says so.
2. Load the `simple-english` skill before you write text that a person reads.
   Examples are a document, a code comment, an error message and a commit
   message. Follow the skill for that text.
3. Load the `go-pedantry` skill before you write or review Go code. Follow it
   where it does not conflict with a rule in this file. A rule here wins.

Questions and feature ideas go to GitHub Discussions of `chromedp/chromedp`,
and bugs go to issues. Do not open an issue for a question.

`CLAUDE.md` holds one line that imports this file, so that Claude Code and
every other agent read the same rules. Edit this file, not that one.

## Which document to read

`docs/decisions/README.md` is the index of every decision. A decision is one
file named by its date and a short title. Read the status of a decision before
you trust it, because a later decision can amend or replace it.

| If you are | Read |
| --- | --- |
| looking for what the repository is, and how to build and run a program | `README.md` |
| asking which program needs the internet and which runs offline | the verification table in `README.md` |
| changing a program for the typed API | `docs/API.md` and `docs/MIGRATION.md` in the `chromedp` repository |
| asking why the programs use the typed API | `docs/decisions/2026-10-03-the-programs-use-the-new-typed-api.md` |
| asking why something is the way it is | the index in `docs/decisions/README.md` |
| recording a decision | `docs/decisions/README.md`, and the section Writing documentation in this file |
| preparing a change as a person | `CONTRIBUTING.md` |
| adding or changing a program | the sections Hard rules and Before you commit in this file |
| running a program against its expected output | the section Verify an offline program in this file |
| making a program read local pages instead of a live site | the section The local test site in this file, and `internal/testsite/README.md` |

## Hard rules

1. Each program is one folder with one `main.go`. The folder can also hold a
   data file or a `README.md` that the program needs.
2. A program uses only the public API of `chromedp` and the packages of
   `cdproto`. Do not copy code from the `chromedp` repository into a program.
3. Keep a program small and readable. A reader must follow it from the top.
   Prefer a plain loop and a plain call over a clever helper.
4. Keep the flags of a program. Do not rename a flag, remove it or change its
   default without asking the maintainer.
5. Start the doc comment above `package main` with `Command <name> is a
   chromedp example demonstrating how to`. Name the task in that first
   sentence. `gen.go` reads it for the table in `README.md`. The sentence must
   not hold a period before its end.
6. A program that reads a live site says in its doc comment which site it
   needs, with the words "It reads <site>." A program that needs a service, a
   file or a terminal that can show images says so too.
7. A program that runs offline serves its own page from a local server. Do not
   make it read the internet. Its doc comment says "It starts a local server and
   needs no internet."
8. Wrap every error with `%w`. Write error messages in lower case, and do not
   start them with "failed to".
9. After you change a doc comment, run `go run gen.go` and commit the new table
   in `README.md`.
10. Do not run a program in a session that has no browser. Run `go build ./...`,
    `go vet ./...` and `go test ./docs/` only.
11. Never put a password, a key or a token in a file or in a message.
12. Every program has the flag `-v` and calls `flag.Parse()`. With `-v`, the
    program adds `chromedp.WithDebugf(log.Printf)` to the options of
    `chromedp.NewContext`. Without `-v`, it prints no protocol messages. Every
    program except `remote` also has the flag `-visible`. With `-visible`, the
    program adds `chromedp.WithVisibleWindow()` and `chromedp.WithKeepOpen()` to
    the same options, and it prints `chromedp.KeptOpen(ctx)` to the standard
    error. A program that builds its own allocator, such as `proxy`, adds
    `chromedp.VisibleWindow` and `chromedp.KeepOpen` to the options of the
    allocator instead. Say this in the doc comment, with the words "Use -v to
    print the protocol messages and -visible to show the browser window and
    leave it open." Use the help text "print the protocol messages" for the flag
    `-v`, and "show the browser window and leave it open" for the flag
    `-visible`. The program `extension` is the exception: it uses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chromedp/examples](https://github.com/chromedp/examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
