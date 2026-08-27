---
trigger: always_on
description: Workspace: mail client under construction. Source material:
---

# notmutt - requirements and architecture

Workspace: mail client under construction. Source material:
`references/neomutt/` (async patches), `references/muttrc/` (live config),
`references/afew/` (per-account MailMover), `references/notmuch/` (notmuch
source), `neovim/` (UI async + Lua reference).

Goal: async mail client. All mail via notmuch.
Architecture constraints, not suggestions.

## The mail concept

Tags = logical model. Folders = physical storage. Every view, filter,
trigger = notmuch query or tag op; folders only for sync-tool
compatibility and physical moves.

### Hard-tag exclusivity (KNOWN PAIN - must be fixed)

Folder tags mutually exclusive: one message, one home (inbox, archive,
deleted, sent, draft, pending, spam). Config enforces by hand: every new
folder tag needs `-newtag` on every older rule + cross-untag rule.
Unacceptable.

Filter engine MUST support declarative exclusive tag groups:
`[tag-groups.folder] tags = ["inbox", "archive", "deleted", "sent",
"draft", "pending", "spam"]`. Applying any member removes the others
automatically. Soft tags in no group - unlimited, coexisting, never
moved. Adding a group tag must not touch existing rules.

### Idempotency

Every filter rule must carry a NOT guard: re-runs touch only new mail
(first backfill ~129k messages takes minutes; steady state must be
cheap).

### Per-account folder priorities

Move destinations resolve per account by existence: candidates in order,
first existing wins, `*` = globs (afew `folder_priorities`). Move engine
must keep this model.

### Lock handling

notmuch lock waits must be capped (`lock_timeout=10`): UI tag ops error
out instead of hanging behind a background index/send.

## Requirements

### R1. notmuch = single source of truth

No own DB. State, flags, threads, search from libnotmuch. Virtual views
= tag queries; folder state derived, never authoritative. ONE derived
store: index cache (R13) - bbolt mirror of the overview query,
revision-keyed, invalidated by lastmod, rebuilt from query output only.
Stale read re-syncs (startup O(changed); full walk only on cache miss
or revision mismatch).

cgo = the runtime backend (record 3); CLI behind `-tags cli` (the F10
escape hatch). The two backends are build-exclusive (`!cli` / `cli`) for
license separation: the cgo variant links GPL libnotmuch and is released
GPL-3.0, the CLI variant links nothing and is released Apache-2.0 (see
docs/licensing.md). Index fill = FULL walk
(record 29), one C pass, no stubs. In-reply-to refs ride the per-thread
fetch (refsfromterms build, record 30). cgo handle read-only; Tag
reopens read-write for the op only (persistent write handle blocks
other notmuch processes).

### R2. Filters and triggers through notmuch + afew

Filter interface must be a boundary: consumes notmuch docs + rules,
emits tag changes. Same contract: hooks/afew pipeline + integrated
engine replacing afew.

MailMover NATIVE (src/filter/mover.go; MailMover.py = reference logic,
never runtime).

Engine owns the FULL pipeline (folder rules, header rules, mover
in-process); muttrc hook + afew = reference shapes, not backends. Folder
rules DERIVED: hard tag candidates -> `folder:"<account>/<candidate>"`
OR-queries with auto NOT-guards; account tags from the folder prefix.
Exclusive groups per the concept section. Header rules = data (query +
add, guards by engine). Conditional rules explicit (delivery
untag-reversal, trash return-to-inbox). Read-only accounts never
classified: no folder tags, no account tag, no header tags, no moves.
Side effects on filter job completion.

Contract per-message (afew shape): snapshot in, tag ops out. Query rules
= data-driven impl; algorithmic filters (SpamFilter, DKIM - later) =
registered Go impls, `[filter.<name>] type = "..."`, unknown type = load
error. Group resolution after every filter. Content filters in-process,
no DB handles, no content out. Mover updates via own notmuch layer, not
a subprocess. DRY-RUN = first-class mode: resolve every target, write
nothing, report what-would-move. Flow: dry-run -> review -> live; first
real-mailbox runs always dry.

### R3. Async read/update with incremental thread views

All reads/updates async; UI never blocks. Query layer must support
thread-type queries (NM_QUERY_TYPE_THREADS): view holds thread objects;
refresh must INSERT new messages into existing visible threads, no full
rebuild. Diff-and-insert, not rebuild. notmutt must do better than
neomutt: async thread loading + diff-and-insert refresh.

### R4. Async send + dialogue state machine

`send_command` async. Dialogue = state machine, state SEPARATE from UI
rendering: fields, attachments, send progress, error, output. Dialogues
PAUSE and RESTART (state survives); composition TABBED (multiple
dialogue states alive concurrently). Send = background job, captured
output kept for review.

Background sync + filter runs must NEVER interrupt an active
composition; anything touching mail state while a dialogue is open must
go through the async layer - never block, never invalidate in place.

### R5. TUI-first, extractable

TUI. Core (dialogue state machines, event handling, rendering
primitives) must be structured so the whole TUI layer extracts into a
standalone library; client = reference consumer. No UI code in core; no

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fishman/notmutt](https://github.com/fishman/notmutt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
