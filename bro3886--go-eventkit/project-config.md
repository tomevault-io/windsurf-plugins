---
trigger: always_on
description: A Go library providing native macOS EventKit bindings via cgo + Objective-C. Exposes idiomatic Go types and a public client API for Calendar events and Reminders. In-process, sub-200ms access — no AppleScript, no subprocesses.
---

# go-eventkit — Go bindings for Apple's EventKit framework

## What is this?
A Go library providing native macOS EventKit bindings via cgo + Objective-C. Exposes idiomatic Go types and a public client API for Calendar events and Reminders. In-process, sub-200ms access — no AppleScript, no subprocesses.

**Repository**: `github.com/BRO3886/go-eventkit`

## Non-Negotiables
- **Conventional Commits**: ALL commits MUST follow [Conventional Commits](https://www.conventionalcommits.org/). Format: `type(scope): description`. Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`. No exceptions.
- **ARC is mandatory**: `#cgo CFLAGS: -x objective-c -fobjc-arc` — without ARC, ObjC objects get released prematurely and EventKit returns empty results or SIGSEGV. This is critical.
- **cgo stays internal**: Public API is pure Go types. No cgo leaking to consumers.
- **JSON bridge format**: ObjC returns JSON via `char*`, Go parses into typed structs. Keeps C interface minimal.

## Architecture
```
go-eventkit/
├── eventkit.go                  # Shared types: RecurrenceRule, StructuredLocation, Weekday, etc.
├── eventkit_test.go             # Tests for shared types and convenience constructors
├── calendar/                    # Public: Calendar event bindings (Phase 1 — COMPLETE)
│   ├── calendar.go              # Go types: Event, Calendar, Client, options, calendar CRUD inputs
│   ├── parse.go                 # JSON parsing/marshaling (platform-agnostic, no build tags)
│   ├── bridge_darwin.go         # cgo wrappers (//go:build darwin) — includes WatchChanges
│   ├── bridge_darwin.m          # ObjC EventKit bridge for EKEvent + watch pipe
│   ├── bridge_darwin.h          # C header
│   ├── bridge_other.go          # !darwin stubs
│   ├── watch.go                 # watchChangesFromFile helper (no build tag, unit-testable)
│   ├── watch_test.go            # 4 unit tests via os.Pipe injection
│   ├── calendar_test.go         # Unit tests
│   └── bridge_mock_test.go      # Mock bridge tests (JSON contract)
├── reminders/                   # Public: Reminder bindings (Phase 2 — COMPLETE)
│   ├── reminders.go             # Go types: Reminder, List, Client, options, list CRUD inputs
│   ├── parse.go                 # JSON parsing/marshaling (platform-agnostic, no build tags)
│   ├── bridge_darwin.go         # cgo wrappers — includes WatchChanges
│   ├── bridge_darwin.m          # ObjC EventKit bridge for EKReminder + watch pipe
│   ├── bridge_darwin.h          # C header
│   ├── bridge_other.go          # !darwin stubs
│   ├── watch.go                 # watchChangesFromFile helper (no build tag, unit-testable)
│   ├── watch_test.go            # 4 unit tests via os.Pipe injection
│   ├── reminders_test.go        # Unit tests
│   └── bridge_mock_test.go      # Mock bridge tests (JSON contract)
├── dateparser/                  # Public: Natural language date parsing (shared by cal + rem CLIs)
│   ├── dateparser.go            # ParseDate, ParseDateRelativeTo, options, all parse functions
│   ├── format.go                # FormatDuration, FormatTimeRange, ParseAlertDuration
│   ├── dateparser_test.go       # 35 test functions (keywords, relative, weekday, tz, DST, etc.)
│   └── format_test.go           # Format/duration test suite
├── scripts/                     # Integration tests (require real EventKit)
│   ├── integration.go           # 35 calendar integration tests (incl. WatchChanges tests 32-35)
│   ├── integration_reminders.go # 34 reminder integration tests (incl. WatchChanges tests 31-34)
│   └── watch-demo/              # Demo: producer + consumer across two processes
│       ├── consumer/main.go     # Watches calendar changes, diffs and prints what changed
│       └── producer/main.go     # Creates, updates, deletes an event with pauses
├── docs/
│   ├── prd/
│   │   ├── go-eventkit-prd.md              # Full PRD with API design
│   │   ├── concurrency-prd.md              # Deferred concurrency improvements (3 phases)
│   │   ├── recurrence-location-prd.md      # Recurrence rules & structured locations (DONE)
│   │   ├── change-notifications-prd.md     # WatchChanges API (DONE — v0.3.0)
│   │   ├── benchmarking-prd.md             # Performance benchmarking (planned)
│   │   └── future-capabilities-prd.md      # Deferred capabilities (10 items)
│   └── research/
│       ├── eventkit-framework-comprehensive.md
│       └── go-concurrency-cgo-eventkit.md
├── journals/                    # Engineering journals (10 sessions)
└── go.mod
```

## Implementation Status
- **Root package** (`eventkit.go`): Shared types — RecurrenceRule, StructuredLocation, Weekday, convenience constructors. Coverage: 100%.
- **Phase 1**: `calendar/` package — COMPLETE. Full event CRUD + calendar container CRUD + recurrence rules + structured locations. Coverage: 56.7%.
- **Phase 2**: `reminders/` package — COMPLETE. Full reminder CRUD + list container CRUD + recurrence rules. Coverage: 54.9%.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BRO3886/go-eventkit](https://github.com/BRO3886/go-eventkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
