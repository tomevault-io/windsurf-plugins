---
trigger: always_on
description: Orientation for anyone — human or agent — writing code in this repo.
---

# AGENTS.md

Orientation for anyone — human or agent — writing code in this repo.

**The single most important rule: search before you write.** The most common
defect in this codebase's history is not bugs, it is the same logic implemented
a fourth time because the author did not know the first three existed. See
[Shared code](#shared-code--check-here-before-writing-a-helper) below.

## Build, test, lint

```
go build ./...
go test ./...                 # ~10s
go test ./... -race           # ~47s; this is what CI runs
go vet ./...
golangci-lint run             # v2.13.1, config in .golangci.yml
gofmt -l .                    # must be empty; enforced in CI
```

Those two timings are wall clock with `-count=1` and a warm build cache on an
8-core Linux box, re-measured at the end of Phase 0 (they were `~21s`/`~44s`
before it, on unrecorded hardware — treat them as a shape, not a target).
The shape is what matters: under `-race`, four packages are ~95% of the run —
`internal/ui` 32s, `cmd/slk` 29s, `internal/cache` 19s, `internal/ui/messages`
16s (they overlap, hence a 47s wall). If you are iterating, run the one package
you are changing; `internal/ui` alone is now longer under `-race` than the whole
suite used to be.

Tests are plain `testing.T`, stdlib only. No testify, no gomock, no golden
libraries. White-box (`package ui`, not `package ui_test`) by convention.

## Architecture in one screen

```
cmd/slk/                composition root: wiring, workspace connection,
                        WebSocket event handling, message fetch/cache pipeline
internal/core/          the ports (service interfaces) the TUI calls, and the
                        values the TUI and cmd/slk exchange through them
internal/ui/            bubbletea App: reducers, mode key handlers, view regions
internal/ui/<widget>/   self-contained sub-models (messages, thread, sidebar,
                        compose, and 13 modal packages)
internal/slack/         Slack Web API + browser-protocol WebSocket client
internal/slack/edge/    edgeapi: conditional revalidation, server-side search
internal/bootstrap/     startup fetch orchestration
internal/cache/         SQLite cache (a cache, not a source of truth)
internal/config/        TOML config
```

**`wiki/Architecture.md` is stale by roughly 7× and describes a service layer
that no longer exists. Do not trust it.** Current structural documentation:

- `docs/superpowers/plans/2026-09-06-architecture-refactor.md` — the active
  refactor: measured baseline, known problems, phase sequence
- `docs/superpowers/plans/2026-05-23-app-go-solid-refactor.md` — the completed
  `app.go` decomposition; establishes the patterns still in use
- `docs/superpowers/specs/` — one design doc per feature

### Invariants worth knowing

- **`internal/ui` does no I/O of its own.** Slack, SQLite, the filesystem, the
  clipboard, the external editor and launching apps all go through the service
  ports in `internal/core`, which `cmd/slk` wires. No `internal/slack`,
  `slackhttp`, `cache`, `config`, `filedl`, `export`, `editor`, `net/http` or
  `os/exec`; `slack-go` only in `blockkit`, as the data it renders. `internal/ui/boundary_test.go`
  enforces this. The boundary is deliberate; do not breach it.
- **`App.Update` routes through a reducer chain**, not a switch. Add behavior by
  adding to a `reducer_*.go` file, not by extending `Update`.
- **Per-mode key handling is a table**, `modeHandlers` in
  `internal/ui/mode_handlers.go`. One `mode_*.go` file per mode.
- **SQLite is a cache.** Slack remains authoritative.

## Shared code — check here before writing a helper

If you are about to write text wrapping, box drawing, list windowing,
scrollbars, date formatting, case folding, or ID formatting: it already exists.

### Text and rendering

| Need | Use |
|---|---|
| Word wrap to a width | `messages.WordWrap(s, limit)` |
| Plain-text line segmentation (grapheme-correct) | `messages.PlainLines`, `messages.DisplayWidthOfPlain`, `messages.SliceColumns` |
| Display width of a string (emoji-aware) | `emoji.Width(s)` |
| Case/accent-insensitive fold for matching | `text.Fold(s)` |
| Slack mrkdwn → plain text | `messages.FlattenMrkdwn`, `messages.FlattenMrkdwnWithUserGroups` |
| Search-term highlighting (ANSI/OSC-safe) | `messages.HighlightSearchTerms`, `messages.SearchHighlightSGR` |
| Extract links from message text | `messages.ExtractLinks` |
| Does message text mention the current user? | `mention.InText(text, selfUserID)` |
| Reaction pill rendering | `messages.ReactionPillText` |
| Date label from a Slack ts | `messages.DateFromTS`, `messages.FormatDateSeparator` |
| mpdm channel name → human name | `slackfmt.FormatMPDMName` |
| Slack permalink parsing | `slackurl.Parse` |
| Emoji shortcode → glyph | `emoji.Sprint`, `emoji.CodeMap`, `emoji.StripSkinTone` |
| Does Block Kit already render the message body? | `blockkit.RendersBody(blocks)`, `messages.BlocksCarryBody(msg)` |
| Current DND state from a Slack API result | `slack.DNDStateFromStatus` |
| Peer custom status, DND and huddle rendering | `ui/peerstatus` (`Status`, glyph/expiry/summary methods); `messages.AuthorStatusSuffix` for author headers |
| Usergroup map helpers | `usergroups.Copy`, `usergroups.Equal`, `usergroups.Display` |

### UI chrome


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gammons/slk](https://github.com/gammons/slk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
