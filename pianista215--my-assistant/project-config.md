---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A Go REST service that decides and serves what an e-ink screen connected to an ESP32 should display. The ESP32 will poll the endpoint hourly; the endpoint renders today's agenda from a single reference Google Calendar next to a right-hand column that itself varies by time of day — the current shopping list at midday, or a visual weather panel for a preconfigured location otherwise — plus the next few days of a weekly menu below both, all from a single reference Google Sheet (calendar and Sheets both via the same one-time-authorized OAuth credential, refreshed unattended — see "Calendar credentials" below; weather comes from Open-Meteo instead, which needs no credential at all). Runs autonomously on a VPS. This is the user's first Go project — prioritize idiomatic, explainable code over shortcuts.

See [README.md](README.md) for usage, endpoint, and binary format details.

## Target hardware

Seeed reTerminal E1001 (GDEY075T7 panel, UC8179 controller): **800×480 px, 4 grayscale levels (2 bits/pixel)**. There's no lightweight standard image format worth adopting for this, hence the custom binary format in `internal/display/codec.go` and the `cmd/preview` CLI to inspect it visually.

## Commands

```bash
go build ./...
go vet ./...
go test ./...
go run ./cmd/server
go run ./cmd/preview --file buffer.bin
```

Unlike other projects of this user, tests **can be run directly here** (`go test ./...`) during implementation — this is a new, small, isolated Go project, without the "don't run tests, give me the command" policy that applies to other repos (that policy belongs to a different, unrelated project).

## Conventions for this project

- **No single-use packages**: don't create an `internal/auth` package just for the token middleware — it lives in `internal/server/middleware.go`, alongside the rest of the HTTP server. If in the future several middlewares are shared across different servers, extracting `internal/middleware` (the `mid` pattern from Ardan Labs Service) would then be justified.
- **`internal/display`, not `internal/eink`**: the package represents *what will be displayed* (image + codec), not the panel driver/firmware. Consistent with the `/api/v1/display` endpoint.
- **Authentication token**: loaded from an environment variable (`AUTH_TOKEN`), with `.env` support in development via `github.com/joho/godotenv`. In production the VPS sets real environment variables (systemd `EnvironmentFile=`), there's no `.env` on the server. Token comparison uses `crypto/subtle.ConstantTimeCompare`.
- **Custom image format**: `internal/display/codec.go` packs 2 bits/pixel with no external standard, meant to minimize memory usage on the ESP32. Any format change must preserve the `Encode`/`Decode` roundtrip and its test.
- **`internal/calendar`, not `internal/googlecalendar`**: represents the domain concept (today's agenda as a list of `Row`), backed by the Google Calendar API client internally. `internal/display` stays agnostic to where its content came from (`Row.TimeLabel()`/`Row.Summary` feed `display.EventLine{Time, Text}`, see "Agenda wrapping" below) — the calendar package owns the formatting logic for its own data, `display` just renders whatever `Section`/`EventLine` it's handed.
- **`internal/shoppinglist`, not `internal/googlesheets`**: same domain-not-API-vendor naming as `internal/calendar`. Reads a fixed spreadsheet's first tab, `A2:A` (no sheet-tab name in the range, so it resolves to the first tab regardless of what it's named; starting at row 2 skips a human-only header row with no special-case code). An item is just its string — no `Row` struct, unlike Calendar, since there's no start/end/reminder classification to model. `parseItems` (pure, tested directly with literal `[][]interface{}` fixtures, same style as `calendar`'s `toRow`/`isVisible`) drops blank rows, non-string cells, and whitespace-only cells.
- **`internal/weeklymenu`, not `internal/sheets2` or similar**: reads the *second* tab of the same spreadsheet as `shoppinglist` — lunch ("comida", rows 2-6) and dinner ("cena", rows 8-12) entries per weekday column (Monday-first, A-G), row 1 being the header. Unlike `shoppinglist`'s tab-agnostic range, a second tab has no implicit A1-notation reference by position, so `FetchWeek` first calls `Spreadsheets.Get` to read `sheets[1].Properties.Title` (the second tab, regardless of what it's actually named) and builds the values range from that title — two API calls per fetch instead of one. The client mirrors `calendar.Client`'s shape (closes over `time.Now().In(loc)` internally so `FetchWeek(ctx)` needs no extra argument), not `shoppinglist.Client`'s, since it has a "today" concept. `parseWeek` rotates the 7 parsed `Day`s to start at today's column (Go's `time.Weekday` is Sunday=0..Saturday=6; `(int(today)+6)%7` converts to this sheet's Monday=0 indexing) and tolerates blanks the same way `shoppinglist.parseItems` does. `Day.Label` is read verbatim from the sheet's header row rather than hardcoded in Go, so the day name's language/format is entirely the spreadsheet's choice.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pianista215/my-assistant](https://github.com/pianista215/my-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
