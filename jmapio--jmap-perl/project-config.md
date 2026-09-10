---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A proxy server that bridges IMAP/CalDAV/CardDAV backends to the JMAP protocol
(RFC 8620/8621, JMAP Calendars, JMAP Contacts). It also supports direct
JMAP-to-JMAP passthrough for backends that already speak JMAP (Cyrus, Fastmail).
One instance serves many users, each with their own backend account.

Email/calendar/contact **conversion logic** lives in three standalone CPAN-style
modules (`Data::JSEmail`, `Text::JSContact`, `Text::JSCalendar`); the **sync and
protocol** code lives here and in `Net::*Talk`.

## The process model (read ARCHITECTURE.md before touching `bin/jmap-proxy.pl`)

`bin/jmap-proxy.pl` is a single-process server that forks children over socketpairs:

- **Parent** — AnyEvent HTTP event loop. Does HTTP routing, request dispatch,
  response callbacks, and child management. **It must NEVER BLOCK.**
- **`__accounts__` child** — owns `accounts.sqlite3` only (account CRUD, tokens, auth).
- **Per-account child** — owns ALL IMAP/CalDAV/CardDAV connections, the per-account
  SQLite file, sync, and JMAP method handling. Blocking JSON read/write loop.

Hard rules that are easy to violate:

- **NEVER in the parent**: `firstsync`, `sync_imap`, `sync_folders`, IMAP connections,
  `JMAP::ImapDB->new()`, `setuser()`, `DBI->connect`, or any per-account DB op.
- **Backfill and sync/jmap MUST run in separate child processes.** The jmap/sync
  worker is keyed by `$accountid`; the backfill worker is keyed by `"$accountid:backfill"`
  (separate fork, same DB). `run_backend_worker` strips a trailing `:[^:]+$` to recover
  the real accountid for DB lookup. The parent drives the loop via `prod_backfill($accountid)`.
  **Never call `$db->backfill()` inside the sync/jmap worker command handler.**
- **Never call `get_user()` outside a transaction** — Perl autovivification of
  `$Self->{t}{user}` leaves a phantom `$Self->{t}={}` and corrupts later transaction state.
  Cache values you need inside the transaction for use in closures.

## Code layout

- `bin/jmap-proxy.pl` — the server (parent + worker loop + cross-account `/copy` orchestration).
- `JMAP/DB.pm` — base DB class (SQLite schema, transactions, sync state, query snapshot cache).
  - `JMAP/ImapDB.pm` (← DB) — IMAP/CalDAV/CardDAV sync. `FastmailDB`, `GmailDB`, `AOLDB` extend it.
  - `JMAP/JmapDB.pm` — standalone, for JMAP passthrough backends.
- `JMAP/Dispatch.pm` — pure, unit-testable dispatch core: `check_accounts` (a missing
  `accountId` is `invalidArguments`, one outside the session is `accountNotFound` — the
  proxy never defaults it), `group_batches` (order-preserving same-upstream batching) and
  `resolve_pointer` (JSON Pointer with JMAP `*` semantics).
  No I/O, no DB — keep it that way; `t/dispatch-*.t` covers it.
- `JMAP/API.pm` — JMAP request handler; dispatches to per-datatype method modules in
  `JMAP/API/` (`Email`, `Mailbox`, `Thread`, `Calendar`, `Contact`, `Submission`,
  `StorageNode`, `MDN`, `Quota`, `Preferences`).
- `JMAP/Sync/` — backend sync drivers (`Standard`, `Gmail`, `Fastmail`, `AOL`, `Common`).
- `JMAP/OAuth/` — OAuth2 signup (`Google`, `Fastmail`, `OIDC`, `PACC`, `PKCE`).
- `JMAP/CredentialStore.pm` — pluggable at-rest encryption for stored credentials.

## Passthrough request dispatch (`_do_jmap_request`)

One JMAP request can span several accounts on different upstreams. The parent groups the
method calls into batches by **upstream key** and runs the batches strictly in order,
threading `createdIds` forward and resolving cross-batch `ResultReference`s itself.

- **Upstream key**: passthrough account → `fp:<cred_fingerprint>`; anything else →
  `imap:<accountid>` (unique, so it never shares a batch).
  `cred_fingerprint` = sha256(apiUrl, username, authType, secret), computed in the worker —
  only the fingerprint, never the raw credentials, reaches the parent.
- **Copy routing**: a `/copy` call whose two sides share one passthrough upstream is
  forwarded natively; otherwise it goes to parent-level orchestration (`_do_copy_call`),
  which shuffles blobs via `fetch_blobs`/`store_blob`.
  The classifier must list **every** `/copy` method `_do_copy_call` handles — a method
  missing from that regex is forwarded to a worker and comes back `unknownMethod`.
  (`t/dispatch-copy-route.t` passes its own stub classifier, so it does **not** catch this.)
- **Server-injected responses**: an orchestrated copy may return several triples (e.g. the
  `Email/set` for `onSuccessDestroyOriginal`). `@responses` is position-indexed, so extras
  go in `@extra_responses` and are appended after all responses. Dropping them silently
  loses the `Email/set` from the client's view.

## Multi-account passthrough (one login, several accounts)

An upstream login often exposes more than its own account (delegated/shared accounts).
Register one proxy account per upstream account, all with the same `username`/`password`
but a different `backendAccountId` (validated against the upstream session's `accounts`).

- `email` is the **proxy-side login** and must stay unique, so it falls back to the
  `backendAccountId` for any account not bound to the primary.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmapio/jmap-perl](https://github.com/jmapio/jmap-perl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
