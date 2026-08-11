---
trigger: always_on
description: Each domain has a service in `internal/{domain}/` following the same shape:
---

# Agent Guide for chroncal

## Service Layer Pattern

Each domain has a service in `internal/{domain}/` following the same shape:

```go
type Service struct {
    db *sql.DB
    q  *storage.Queries
}

func NewService(db *sql.DB, q *storage.Queries) *Service {
    return &Service{db: db, q: q}
}
```

Core data services:
- **event** - CRUD, search, export, recurrence-aware queries, soft-delete/restore/purge
- **todo** - CRUD, search, completion, soft-delete/restore/purge
- **journal** - CRUD, search, soft-delete/restore/purge
- **calendar** - CRUD, color management, remote-link metadata
- **alarm** - Check due alarms, fire, dismiss, snooze
- **recurrence** - Expand recurring events/todos/journals, handle overrides
- **trash** - Mixed soft-delete view across event/todo/journal (list, restore, purge)

Integration / infrastructure packages (these do NOT follow the `NewService`
shape above — constructors and wiring vary per package):
- **sync** - CalDAV sync engine, conflict detection and resolution (`NewService` with extra dependencies)
- **caldav** - Low-level CalDAV client (discovery, REPORT, PROPFIND, VFREEBUSY) — `NewClient`
- **freebusy** - Local free/busy computation plus remote CalDAV query — plain functions (`Compute`)
- **auth** - Credential storage (OS keyring, optional plaintext), OAuth2 PKCE — plain functions
- **maintenance** - Background purge loop for soft-deleted rows — `NewPurger`
- **notify** - Desktop notifications plus SMTP email for EMAIL alarms — plain functions (`Display`, `Audio`, `Email`)
- **retry** - HTTP retry/backoff helpers shared by sync and caldav — plain functions

Models live in `internal/{domain}/model.go` (e.g., `event.Event`) and shared models in `internal/model/` (e.g., `model.Alarm`, `model.Attendee`).

CLI commands live in `cmd/chroncal/`, one file per resource group. Each exports a `Command()` function returning a `*cobra.Command`. Commands use `resolveEvent()` / `resolveTodo()` / `resolveJournal()` helpers to resolve references by ID, UID, or UID+recurrenceID.

## Storage Layer

- Hand-written files in `internal/storage/`: `connect.go` (DB setup), `nullable.go` (helpers), `query_builder.go` (dynamic WHERE construction), `scan_helpers.go` (row scanners), `events_dynamic.go` and `todos_dynamic.go` (filtered query methods), `xprop_helpers.go` (alarm X-property attach/replace shared by event and todo services). Everything else is sqlc-generated and will be overwritten by `make generate`.
- The dynamic query files replace sqlc's `arg = '' OR column = arg` pattern with runtime WHERE clause construction so SQLite can use indexes. Queries use `SELECT *`, so if a migration adds columns to `events` or `todos`, only update the scan functions in `scan_helpers.go` to match.
- **Never edit `*.sql.go` files or `db.go` or `models.go` directly.**
- Add new queries to `db/queries/*.sql`, then run `make generate`.
- After schema changes: add a migration to `db/migrations/`, update queries, then regenerate.
- Transaction pattern: `q.WithTx(tx)` inside a transaction.

## Gotchas

### Database
- Case-insensitive Unicode search goes through FTS5 (`unicode61 remove_diacritics 2` tokenizer); see the `*_fts` virtual tables in `db/migrations/`. There is no custom `lower_unicode` SQLite function — a stale registration that no query referenced was removed. Do not reintroduce `strings.ToLower`-backed folding: it is simple case folding only and would not match the FTS tokenizer's diacritic-insensitive behavior.
- `backfillAlarmUIDs` in `connect.go` assigns UUIDs to alarms from the pre-UID schema. Runs on every startup, no-ops when all alarms have UIDs.
- SQLite pragmas set in `connect.go:Open()`: WAL mode, foreign keys ON, 5s busy timeout, synchronous=NORMAL.

### Recurrence
- Recurring events are stored as a single row with `recurrence_rule`.
- Overrides are separate rows with the same `uid` but a non-empty `recurrence_id`.
- EXDATEs and RDATEs are comma-separated RFC 3339 strings.
- Expansion happens at query time via `recurrence.ListExpandedEvents()`.
- Half-open time ranges everywhere: `[start, end)`.

### Alarms
- Triggers are RFC 5545 duration strings (`-PT15M` = 15 minutes before).
- Absolute triggers use RFC 3339.
- State is tracked in `alarm_state` / `todo_alarm_state` tables (fired_at, acknowledged_at, snooze_until).
- Alarms older than 24h are skipped (`alarm.StaleThreshold`).
- Repeat logic: additional firings at `Duration` intervals up to `Repeat` count.

### iCal Round-Trip
- UID is required for round-trip fidelity.
- `recurrence_id` distinguishes overridden instances.
- Transient fields (Alarms, Attendees, etc.) are populated for export but not stored in the main event/todo tables.
- Duration can be expressed as either DTEND or DURATION (RFC 5545).
- Timezones are preserved via the `timezone` column and the `timezones` table.

### Time Handling
- All database times are RFC 3339 strings in UTC.
- Go code uses `time.Time` with `time.UTC`.
- All-day events have time component 00:00:00.

### TUI Buttons
- Exactly two variants: `Button` (neutral default) and `ButtonDanger`
  (destructive). No Primary, no Secondary, no Ghost.
- `ButtonDanger` at rest shares the same pill and background as
  `Button`; only the *label* is bold red (`Theme.Error`). On focus

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DouglasdeMoura/chroncal](https://github.com/DouglasdeMoura/chroncal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
