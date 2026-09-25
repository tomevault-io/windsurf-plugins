---
trigger: always_on
description: Single Rust binary for browser automation via CDP, built for AI agents. 44 subcommands
---

# chrome-agent v0.16.0

Single Rust binary for browser automation via CDP, built for AI agents. 44 subcommands
(`chrome-agent --help`), 30.9K lines of Rust in `src/` across 91 files (blank and comment-only lines excluded),
one regex crate (`regex-lite`, for `assert --matches`), 3 MB binary.

## Product direction

Read [the mission](docs/mission.md) and [roadmap](docs/roadmap.md) before extending the product.
The target is autonomous discovery of web capabilities, with verified recipes that agents can
reuse, repair and share through a GitHub catalogue or private sources. The runtime below is
the shipped foundation. `discover` now persists caller-driven experiments and exports local
candidates. The first [isolated discovery experiment](docs/experiments/discovery-2026-09-13.md)
demonstrates discovery and reuse on one synthetic site, but rejects the generated recipe for
false completeness claims. M1 and catalogue acceptance remain open. See [the implemented protocol](docs/discovery.md).
Discovery is driven by the calling agent; no embedded model/provider layer is planned. The
first public catalogue covers read and extraction recipes; external-write recipes come later.
Prioritize work that enables that lifecycle. Existing macros and Python examples establish
execution behavior; they are not a sandbox for recipes from untrusted sources.

## Architecture

```
CLI (clap) → CDP Client (WebSocket) → Chrome
```

| Module | Role |
|--------|------|
| `src/cli.rs`, `src/cli_actions.rs` | clap definition (`Cli`, `Command`, one variant per verb) and the per-verb subcommand enums (`MacroAction`, `EmulateAction`, `AssertWhat`, `WebmcpAction`, `DaemonAction`) |
| `src/discovery.rs`, `src/discovery_cmd.rs`, `src/discovery_store.rs` | Caller proposals, revision/budget/history validation, shared dispatcher execution, durable reservations, atomic private files and local candidate export. |
| `src/main.rs`, `src/run.rs`, `src/run_helpers.rs`, `src/connect_cli.rs` | the binary entry point; CLI dispatch on `Command` — each arm builds typed args from clap, calls the SAME `pipe_dispatch::dispatch_*` pipe and batch call, and renders the answer; shared output/error handling and `connect_page` (8-attempt retry); resolving one invocation's browser + page connection |
| `src/page_ctx.rs` | `PageCtx`: the two clients, the store, the three names that locate a page in it, and the two global flags, in one struct — so a dispatcher takes three parameters instead of eleven |
| `src/render.rs` | text-mode renderer: the `value:` / `values lost:` / `verdict:` / `next:` lines, colour only on a tty |
| `src/pipe_command.rs` | the pipe/batch protocol as types: one `deny_unknown_fields` struct per verb, so a mistyped key is an error instead of a silently ignored one |
| `src/pipe.rs`, `src/pipe_command.rs`, `src/pipe_validate.rs`, `src/pipe_dispatch*.rs`, `src/pipe_report.rs`, `src/pipe_emulation.rs` | pipe mode (persistent connection, JSON stdin/stdout); typed per-verb protocol plus cross-field validation; the dispatchers shared by pipe, batch and CLI batch, plus `dispatch_assert` and `run_batch`; `mutates_page` + `attach_change_report`; strict JSON emulation parsing |
| `src/macros*.rs` | macro file format and store (`macros.rs`), session history → macro (`macros_record.rs`), guarded execution (`macros_run.rs`), the `macro list/show/record/run` surface (`macros_cmd.rs`) |
| `src/cdp/`, `src/setup.rs` | WebSocket transport, message correlation, CDP types, the input-event deadline (`send_input`), `ensure_foreground`; console interceptor injection and 7 stealth patches |
| `src/element.rs` | uid resolution, fill, type, press, the settle machinery, `js_exception`. Owns `SECRET_FIELD`, the one predicate deciding whether a value may be printed |
| `src/element_pointer.rs` | the pointer path: `PointerVerb`, `aim_and_dispatch` and the one aim rule both verbs share, native mouse/touch dispatch, the JS fallback when there is no box to aim at, `hover`. Split from element.rs for the 1000-line cap, re-exported via `pub use` — it is the half of `element` that moves when `hit_test` moves |
| `src/element_controls.rs` | select, check/uncheck, upload, drag. Owns `CHECKABLE_PROBE` and `SELECT_READ`, which `assert` reads through |
| `src/element_selector.rs`, `src/element_ref.rs`, `src/read_back.rs` | CSS-selector actions (click/dblclick/fill/focus); the `ElementRef` abstraction over CDP node identity; the `value:{requested,actual,verbatim}` object every read-back verb puts on its response |
| `src/hit_test.rs`, `src/hit_test_report.rs`, `src/geometry.rs` | where a mouse event will land (probe, settle loop, `Delivery` classifier, `--on-intercept`) and what the response says about it (`Dispatched`, `Unaimable`, `Refused`, carrying `intercepted_by`/`verdict`/`next`); box-model → screenshot clip math |
| `src/snapshot.rs`, `src/snapshot_render.rs`, `src/snapshot_secret.rs` | `take_snapshot` and `take_views` over `Accessibility.getFullAXTree`; the pure renderer (compact text, stable uids, role filter, depth limit, subtree focus); redaction of secret field values (`MARKER` = `<redacted>`), decided by asking the page since secret-ness is a property of the element |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sderosiaux/chrome-agent](https://github.com/sderosiaux/chrome-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
