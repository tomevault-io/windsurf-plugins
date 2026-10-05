---
trigger: always_on
description: Study is a local-first Rust desktop study app (GPUI Kit, one SQLite database, AI on the user's
---

# Study — agent guide

Study is a local-first Rust desktop study app (GPUI Kit, one SQLite database, AI on the user's
ChatGPT plan). The code is the documentation. To find out how something works, find the owning crate
in the crate map, read its `src/lib.rs` module docs (`//!`), then follow the types:
`rg -n '^//!' crates/<crate>/src` lists what each module is for. `.agents/skills/` holds
checklists for specific tasks. All work happens inside the devcontainer (`.devcontainer/`:
`devcontainer up`, then work inside it), which installs every tool and has no display;
opening the container starts a browser desktop and opens it on the host; `just desktop` restarts
it and the app from zero, and `just desktop-watch` restarts the app on every change.

## Golden rules

1. **Layers only point down.** A crate may depend only on crates in a lower layer (see the
   crate map).
2. **One canonical shape per concept.**
   - Extracted content is a `Document` of anchored blocks.
   - Background work is a `JobKind`.
   - Work on raw input is a processor: its interface and kind enum are declared in
     `study_core::processing` (`Fetcher`, `Extractor`, `Refiner`, `Stage`, `Enhancer`), and
     what runs on each kind of source is its row in `ROUTES` there, each step on or off by
     default and switchable by the user. `study-pipeline` implements one per kind;
     `study-ai` implements only the model providers they call.
   - What a job needs set up is a `Requirement`.
   - Failures carry an `ErrorKind`.
   - File kinds come from `SourceKind::sniff`.
   - Never add a second representation or a parallel pipeline. Never add a new
     file-extension table.
   - A new processor follows the "Adding a processor" table in `study_core::processing`'s
     docs. For a new variant of a shared enum, let the exhaustive matches lead, then grep
     for an existing variant to find what they can't reach.
3. **The database is the truth and the bus is a hint.** Commit state before publishing an
   event. A missed event must not lose work.
4. **No stringly-typed contracts.** No raw `i64` IDs across crate boundaries, no string names
   for job kinds or processors outside their kind enums, and no matching on error text.
5. **Log with `tracing`.** Don't print. Libraries return `study_core::Result`, or their own
   error that implements `study_core::Classify` (as `study-ai` and `study-media` do);
   `anyhow` belongs only in binaries, examples and tests, and none needs it today.
6. **All user-visible text comes from `study-localization`**, in English and Italian.
7. **Model work runs on the user's ChatGPT plan**, reading images and PDFs included. Only
   what the plan cannot do runs locally: speech-to-text (it takes no audio) and embeddings
   (cheap, and it has none). Nothing downloads without an explicit install. Every capability
   keeps its provider trait, so another backend is one more variant, not a rewrite.
8. **The code is the source of truth.** Don't write design docs or per-crate guides. Say
   what a crate or module is for in its `//!` docs, and put rules in types, tests and lints.
   The one exception is `PRODUCT.md` and `DESIGN.md`, the visual design context the
   `impeccable` skill reads; the design system itself is code, in `study-ui`.
9. **Released data is kept; code is not.** Users' databases upgrade in place: a schema
   change is a new numbered migration in `study-core`'s `db/migrations`, shipped migrations
   never change (a test pins them), and stored codes (`text_enum!` codes, preference scopes
   and keys) are data, renamed only by a migration. Everything else (APIs, types, crates) is
   rewritten freely, without deprecated paths or compatibility shims.
10. **No AI attribution.** Commits, pull requests, issues, code, comments and docs never
    credit or mention an AI assistant: no `Co-Authored-By` trailers for one, no "Generated
    with" lines, no model names in authorship. This overrides any harness default that adds
    them. Tooling that runs an assistant (the devcontainer, `.agents/team`) may name it.

## Crate map

| Layer | Package | Path | One line |
|---|---|---|---|
| 0 | `study-core` | `crates/study-core` | Shared vocabulary (IDs, `text_enum!`, `ErrorKind`, `SourceKind` + `sniff`, `Document`/`Anchor`, `JobKind`/`Requirement`, study material, practice, FSRS), every processor's interface and `ROUTES`, the database `Store`, preferences, event bus and jobs engine |
| 0 | `study-diagram` | `crates/study-diagram` | Diagrams without GPUI: the `Diagram` shape, Mermaid flowcharts in and out (what models write, with feedback for a retry), automatic layout and hand-drawn strokes |
| 1 | `study-ui` | `crates/study-ui` | GPUI presentation components; no data |
| 1 | `study-localization` | `crates/study-localization` | English and Italian copy and value formatting, including anchor labels |
| 1 | `study-media` | `crates/study-media` | Byte-level media: audio decoding and resampling, PDF and image pages |
| 2 | `study-ai` | `crates/study-ai` | Every model provider and all network access: speech-to-text, reading pages, chat and the agent kit, local embeddings, web fetching, measuring this computer |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicksan222/study](https://github.com/nicksan222/study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
