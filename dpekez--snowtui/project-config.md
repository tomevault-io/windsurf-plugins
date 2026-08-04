---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A terminal UI (Bubble Tea / Elm architecture) for browsing and editing ServiceNow records via the Table API and Aggregate (Stats) API. Single Go binary, no backend of its own — it talks directly to a configured ServiceNow instance.

## Commands

```bash
go build -o snowtui .     # build
go vet ./...              # static checks
go mod tidy                # sync go.mod/go.sum after dependency changes
./snowtui [config.yaml]    # run; defaults to ./config.yaml if no path given
```

There is no test suite (`*_test.go` files do not exist yet); `go build` and `go vet` are the only current correctness checks. If you add tests, standard `go test ./...` applies.

Runtime requires a `config.yaml` (gitignored, copy from `config.yaml.example`) with at least one instance. This is a real ServiceNow connection — do not fabricate or guess instance URLs/credentials, and don't assume the contents of the local `config.yaml`.

Since this is a TUI, verifying a UI change means actually driving the program (Bubble Tea apps can't be meaningfully checked by reading code alone) — e.g. run it in a real terminal, or drive it headlessly via a PTY + keystroke script if one is available, rather than declaring a fix correct from a diff.

## Architecture

### Navigation: one root state machine, dumb sub-views

`ui/app.go`'s `AppModel` is the only `tea.Model` registered with Bubble Tea. It holds a `viewState` enum (`stateInstanceSelect` → `stateTableSelect` → `stateRecordList` ⇄ `stateGroupView` / `stateRecordDetail`) and one sub-model per state (`InstanceView`, `TableView`, `ListView`, `GroupView`, `DetailView`, in `ui/*_view.go`).

Sub-views never navigate directly or know about each other. Instead they emit intent as typed `tea.Msg` values (defined in `ui/messages.go`: `instanceSelectedMsg`, `tableSelectedMsg`, `recordSelectedMsg`, `newRecordRequestedMsg`, `groupViewRequestedMsg`, `groupSelectedMsg`, `goBackMsg`, ...). Only `AppModel.Update`'s top-level switch reacts to these, mutates `state`, and constructs/initializes the next sub-view. When adding a new screen or cross-view transition, follow this pattern rather than reaching into another view's fields.

`AppModel` also carries state that's inherently cross-view: `listFromGroup`, `groupFilterLabel`, and `savedListView` exist purely so "esc" from a group-filtered list returns to the GroupView (and restores the pre-group `ListView`) instead of falling through to table selection. This bookkeeping lives in `app.go`, not in `ListView`/`GroupView` themselves.

### Each sub-view is a small local state machine

Every `*_view.go` file repeats the same shape: a `Model` struct, a private `<name>Mode` enum for what that view is currently doing (e.g. `ListView`'s `listLoading`/`listBrowsing`/`listSearching`/`listConfirmDelete`/`listError`, `DetailView`'s `detailReady`/`detailEditingField`/`detailConfirm`/`detailSaving`/...), and an `Update`/`View` that switches on that mode. Follow this local-mode-enum idiom for new view behavior instead of adding ad hoc booleans.

All network I/O is dispatched as a `tea.Cmd` closure that captures the needed fields (client, table, query, ...) by value before returning, so the async call never touches the view struct directly — it comes back as a message (`recordsLoadedMsg`, `groupStatsLoadedMsg`, `errMsg`, `mutationErrMsg`, `recordSavedMsg`, `recordDeletedMsg`) that `Update` applies on the main loop. Mutations (create/update/delete) report failure via `mutationErrMsg` specifically so the current form/list stays on screen instead of being torn down like a fatal `errMsg` load failure.

### `api/client.go`

One `Client` per selected instance (constructed in `app.go` on `instanceSelectedMsg`), authenticated with either the `x-sn-apikey` header or HTTP Basic — never both (`APIKey` takes precedence when set). Every failed or non-2xx response is dumped verbatim (including full HTTP response) to `snowtui.log` via `dumpErrorToLog` — that log is the first place to check when an API call misbehaves, since the TUI itself only surfaces a short error string.

Two error-handling styles currently coexist: `GetRecords`/`GetRecord`/`GetGroupStats` switch on `resp.StatusCode` inline, while `CreateRecord`/`UpdateRecord`/`DeleteRecord` share `parseAPIError`, which parses ServiceNow's `{"error":{"message","detail"}}` JSON body. Match whichever style the function you're editing already uses.

ServiceNow-specific behavior baked into the client (see README's "API Reference" section for the full endpoint/param/header table):
- Reads use `sysparm_display_value=true`; `GetGroupStats` instead uses `sysparm_display_value=all`, which is required (not `true`) so `groupby_fields` includes both `value` (used to build the drill-down query) and `display_value` (used as the label) — `true` collapses them into just the display text.
- Writes additionally set `sysparm_input_display_value=true` so submitted field values are interpreted as display values, matching what the UI shows/edits.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dpekez/snowTUI](https://github.com/dpekez/snowTUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
