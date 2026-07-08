---
trigger: always_on
description: SMS-based personal assistant for ADHD management. Everything is a **nag** on a single **today list**: you capture items (with `.. `), each gets an **expire time**, and you're reminded on a **Zeno's-paradox cadence** — per-item nags that accelerate as each item's expire time approaches — then, once an item is overdue, a jittered ping every 30–45 minutes until it's done. Also handles Gmail action-item extraction, scheduled basement-light flashes, and morning briefings. Built with FastAPI, PostgreS
---

# ADHD SMS Bot

SMS-based personal assistant for ADHD management. Everything is a **nag** on a single **today list**: you capture items (with `.. `), each gets an **expire time**, and you're reminded on a **Zeno's-paradox cadence** — per-item nags that accelerate as each item's expire time approaches — then, once an item is overdue, a jittered ping every 30–45 minutes until it's done. Also handles Gmail action-item extraction, scheduled basement-light flashes, and morning briefings. Built with FastAPI, PostgreSQL, Twilio, OpenAI GPT-4o, and Gmail IMAP.

> Reminders and exercise tracking were removed — the system is nags-only. The `Reminder`/`ExerciseLog` models and their intents no longer exist; their tables linger as legacy.

## Architecture

Four Docker services (`docker-compose.yaml`):
- **api** (port 8000): FastAPI SMS webhook (`/sms`) receives Twilio POSTs
- **scheduler**: Background loop (every `TICK_SECONDS=60`s) fires due items + Gmail sync every 30min
- **ui** (port 8081): Web dashboard for viewing/deleting items
- **db**: PostgreSQL 16

## Database Tables

| Table | Purpose |
|---|---|
| `nag_schedules` | The core model — every today-list item (user-created + Gmail action items) |
| `scheduled_flashes` | One-time basement-light flashes ("flash lights at 9pm") |
| `pending_confirmations` | Stores confirmation/follow-up requests, incl. `set_deadline` and undo (10-min TTL) |
| `processed_emails` | Tracks Gmail Message-IDs to prevent re-processing |
| `app_state` | Key-value scheduler state (e.g., "briefing_last_sent_date", "calendar_last_imported_date" for the daily calendar import, "last_nag_sent_at" for the global nag-rate gate, "user_context" for the latest location/intent. Legacy keys from the retired digest model — "next_digest_at", "next_overdue_ping_at", "pending_burst", "burst_last_sent" — may linger but are unused) |
| `sms_log` | Full audit log of all inbound/outbound SMS |
| `checklists` | Named, one-off checklists (created via `#newlist`) |
| `checklist_items` | Items belonging to a `checklists` row (ordered by `position`) |

Legacy tables still in DB but unused by code: `reminders`, `exercise_log`, `action_items`, `recurring_schedules`, `daily_checklist_items`.

## Key Concepts

### Light flashes (`app/models.py: ScheduledFlash`)
"flash lights at 9pm" → `flash_lights` intent → a `ScheduledFlash` row with `fire_at`. The
scheduler's `fire_due_flashes` triggers `_flash_basement_light()` (IFTTT webhooks) at the
time, marks it `done`, and sends a short confirmation SMS. This is the only surviving piece
of the old reminder/event-pair light-flash behavior.

### Nags (`app/models.py: NagSchedule`)
Nags are the unified model for both user-created nags and Gmail-extracted action items. Each item has an **expire time** (`deadline_at`) and nags itself on a **Zeno's-paradox cadence**: each successive nag waits a random 25–50% of the time remaining until the expire time, so reminders accelerate (with jitter) as the deadline nears, then flatten to a jittered every-30–45-minutes ping once overdue.

**Timing concepts:**
- **Expire time** (`deadline_at`): when the item is "due." For one-shots, defaults to 11 PM; for recurring, computed each cycle as the cron start + `deadline_offset_minutes`.
- **Recurrence** (`cron_expression` + `repeating`): how often cycles repeat (e.g., "weekdays at 9am").

**Nag lifecycle (state machine across `scheduler.py` fire functions):**
1. **Dormant** (`active_since=NULL`): waiting for `next_nag_at` to arrive
2. **Cycle start** (`fire_cycle_starts`): when `next_nag_at` passes, sets `active_since=now`, `nag_count=0`, `snooze_count=0`; for recurring sets `deadline_at=now+deadline_offset_minutes`; then points `next_nag_at=now` so the first Zeno nag fires the same tick. (While active, `next_nag_at` is the per-item Zeno send clock; the next cron cycle is recomputed fresh on check-off / overnight rollover.)
3. **Active**: `fire_due_nags` sends Zeno-spaced nags until the item is checked off or rolled over
4. **Check-off / completion**: ends the item for today (recurring → next cycle; one-shot → `status="deleted"`)

**Zeno cadence** (`scheduler.py: fire_due_nags`, `_compute_deadline_interval`):
- Every tick, `fire_due_nags` selects active items whose `next_nag_at` has arrived, sends one nag per item (a rich GPT-written deadline message via `openai_client.generate_deadline_nag_message`, plain fallback), and advances each `next_nag_at` by its Zeno interval

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IsaacRay/automatic](https://github.com/IsaacRay/automatic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
