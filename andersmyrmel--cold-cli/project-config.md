---
trigger: always_on
description: Agent-first CLI cold email sequence engine in Go. See ARCHITECTURE.md for full design.
---

# cold-cli

Agent-first CLI cold email sequence engine in Go. See ARCHITECTURE.md for full design.

## Quick Reference

- **Language:** Go
- **CLI framework:** Cobra
- **Database:** SQLite by default via `modernc.org/sqlite`; Postgres supported via `COLD_CLI_DATABASE_URL`
- **External dependency:** gws CLI (subprocess calls for Gmail API)
- **Config/data dir:** `~/.cold-cli/`

## Project Structure

```
cmd/cold-cli/main.go     — CLI entry, Cobra command definitions
internal/                 — single flat package, all application logic
  db.go                   — schema bootstrap, SQLite migrations, indexes
  store.go                — dialect-aware store open, backend selection, tick locking
  sql_runner.go           — cross-dialect query execution + placeholder rebinding
  models.go               — structs (Account, Lead, Campaign, ScheduledSend, Event)
  tick.go                 — tick engine (dialect-aware lock, poll, send loop)
  scheduler.go            — eager schedule computation, variant assignment, round-robin
  gws.go                  — GWSClient interface + real subprocess implementation
  send.go                 — RFC 2822 message construction, threading headers
  reply.go                — reply/bounce detection, In-Reply-To header matching
  template.go             — {{placeholder}} replacement, alias resolution, unresolved stripping
  csv.go                  — lead CSV import, BOM stripping, field validation
  config.go               — YAML config loading
  campaign.go             — campaign CRUD, preview, rendered preview, daily limit warnings
  account.go              — account CRUD, update, domain diagnostics
  lead.go                 — lead pause/resume/blacklist/list, campaign remove-lead
  stats.go                — campaign/step/variant/lead stats, event log
```

## Key Design Decisions

These are settled — do not revisit without explicit instruction:

1. **Eager scheduling** — all sends stored in `scheduled_sends`; then deterministically rebalanced across active/draft campaigns sharing an account. Do NOT use lazy/rolling `next_send_at` on campaign_leads.
2. **GWSClient interface** — gws interaction goes through an interface (`SendEmail`, `ListMessages`). Real impl calls subprocess. Tests use a mock.
3. **Template rendering** — `strings.ReplaceAll` for `{{placeholder}}` substitution with alias resolution (`name` → `first_name`, etc.). Unresolved variables stripped at send time (not sent literally). No Go `text/template`. No template engine.
4. **Daily limits** — count from events table (`SELECT COUNT(*) ... WHERE type='sent' AND timestamp >= today`) and apply them through shared rebalance logic used by preview, warnings, and tick. No mutable `sends_today` counter on accounts.
5. **Account rotation** — round-robin at schedule time. All steps for one lead use the same account (thread continuity).
6. **Thread management** — after step 1 send, backfill `thread_id` and `parent_message_id` onto all remaining `scheduled_sends` for that lead+campaign.
7. **Error isolation** — gws send failure marks that one `scheduled_sends` row as `'failed'` and continues. Never crash the whole tick. Emails with empty subject/body after rendering are also marked `failed` (not sent).
8. **Status semantics** — `skipped` = auto-cancelled (reply/bounce/domain-reply). `cancelled` = user action (pause/blacklist). These are distinct.
9. **Tick locking** — SQLite mode uses flock/fcntl on `~/.cold-cli/tick.lock`; Postgres mode uses an advisory lock on a dedicated connection. Keep the semantics aligned.
10. **Validation at creation** — template placeholders validated against lead CSV at campaign creation with alias resolution and Levenshtein "Did you mean?" suggestions. Unresolved vars stripped at send time as a safety net.
11. **Workspace ownership** — `cold-cli` is the source of truth for account/campaign ownership. Accounts and campaigns carry `workspace_id`, defaulting to `default` for backward compatibility; there is no separate app-side account mapping to maintain. Use `--workspace <id>` or `COLD_CLI_WORKSPACE_ID` when adding inboxes or campaigns for hosted/multi-brand setups; do not rely on email-domain inference as the access boundary. For hosted dashboards or multi-tenant control planes, always pass the intended workspace explicitly so campaigns do not accidentally land in `default`.

## Testing

- Use real SQLite (`:memory:`) in tests for the main behavioral suite. Do NOT mock the database.
- Add focused Postgres boundary tests around dialect seams (rebinding, clone/tick/store lock paths) when touching cross-dialect behavior.
- Mock only the `GWSClient` interface.
- Test scheduler, template rendering, reply matching, bounce parsing as pure functions.
- Every codepath needs: happy path + key error branches.

## Build & Run

```bash
go build -o cold-cli ./cmd/cold-cli
go test ./...
```

## scheduled_sends Status Values

```
pending   → waiting to send
sent      → successfully sent via gws
failed    → gws send failed (error stored in error_message column + events table)
skipped   → auto-cancelled (reply/bounce/domain-reply detected)
cancelled → user-cancelled (pause/blacklist)
```

## Data Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andersmyrmel/cold-cli](https://github.com/andersmyrmel/cold-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
