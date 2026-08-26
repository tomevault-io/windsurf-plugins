---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Automated email campaign system for **AniOne** (anione.me) that sends marketing emails via the **Resend API** using **Broadcasts** to managed **Segments**. Deployed on **Railway** as a daily cron job.

## Architecture

- **`deploy_new_year.py`** — Active campaign dispatcher (runs daily via Railway cron). Uses a date-keyed `CAMPAIGN_MAP` to determine which local template folder and which Resend Segment to send for a given day. Reads the template's HTML/text/Subject from disk and sends it as a Resend **Broadcast** (create + send in one API call). The schedule is a two-wave + paid-finale design across **five segments (A–E)** — see **Monthly Campaign Structure** below.
- **`sync_contacts.py`** — Pushes the `email/` CSV lists into the five Resend Segments (`RESEND_SEGMENT_A`…`E`), creating each unique contact **once with the full set of segments it belongs to** (so a paid user in both Segment A and E is one contact, billed once). CSVs remain the source of truth; run once per month after updating lists. `--create` creates all five segments and prints IDs; `--fresh` deletes all contacts first — **required whenever segment membership changes**, since Resend only sets a contact's segments on creation. (No bulk-import API — contacts are created one-by-one, throttled under 5 req/s.)
- **`send_daily.py`** — Legacy multi-phase campaign script (Christmas campaign). Sends to VIP list first, waits 90 minutes for IP warmup, then sends to cold list. Run manually with `--day N` flag.
- **`scrub_lists.py`** — Interactive email list cleaner. Validates emails (syntax + MX records via `email_validator`), filters bot patterns (+ aliases, excessive dots). Reads from and writes `CLEANED_` prefixed files to `email/` folder.
- **`remove_duplicates.py`** — Deduplicates CSV email lists with Gmail normalization (dot/plus-alias handling). Hardcoded `INPUT_FILE`/`OUTPUT_FILE` paths must be edited per use.
- **`test.py`** — Scratch file for formatting raw email lists; not a test suite.
- **`email/`** — CSV files containing user email lists segmented by signup date and payment status. Column header is `email` (lowercase) in newer files.

## Monthly Campaign Structure & Contact CSVs

Each month's campaign runs ~26 days in three phases on the daily cron (`CAMPAIGN_MAP`), with **all contacts pre-loaded into five Resend segments A–E at once** — no rotation/clearing (fits the **$80 / 10,000-contact** marketing tier):

| Phase | Dates of month | Sends to (alternating) | Audience |
|---|---|---|---|
| **Wave 1** | 1–12 | Segment A → B | free list 1 + 2, **plus paid riding along in Segment A** |
| **Wave 2** | 13–24 | Segment C → D | free list 3 + 4 |
| **Paid token drops** | 13, 17, 21 | Segment E | paid only — 3 "care package" gift emails (20 image tokens each) filling the paid gap |
| **Paid finale** | 25–26 | Segment E | paid only (2 dedicated finale templates) |

The 6 wave templates (`day-1,3,5,7,9,10`) rotate A→B in Wave 1, then again C→D in Wave 2; the finale uses `finale-1` (gift) + `finale-2` (sale) sent to Segment E. The 3 paid drops use `drop-1/2/3` (codes `DROP1/2/3`). Segment IDs are env vars `RESEND_SEGMENT_A`…`RESEND_SEGMENT_E`.

**Multiple sends per day:** a `CAMPAIGN_MAP` date may map to a **single send dict OR a list of send dicts** — e.g. Jul 13/17/21 each fire a Wave 2 free send *and* a paid token-drop. `main()` dispatches each send for the day.

**CSVs to prepare each month: 5 (4 free + 1 paid).** Wired in `deploy_new_year.py`:

| CSV (in `email/`) | `GROUP_*_FILES` → segment | Phase |
|---|---|---|
| free list 1 | `GROUP_A_FILES` (with paid) → A | Wave 1 |
| free list 2 | `GROUP_B_FILES` → B | Wave 1 |
| free list 3 | `GROUP_C_FILES` → C | Wave 2 |
| free list 4 | `GROUP_D_FILES` → D | Wave 2 |
| **paid** | `GROUP_A_FILES` **and** `GROUP_E_FILES` → A + E | Wave 1 + finale |

The paid list is one file referenced in two segments; `sync_contacts.py` creates each contact **once with the full set of its segments**, so paid is billed once.

**Sizing rule (hold-all-at-once):** the only limit is total unique contacts ≤ the plan's contact cap:
`free1 + free2 + free3 + free4 + paid ≤ CAP` (paid counted once). Equal free lists keep daily volume even, but exact sizes can vary as long as the sum fits.

**Example at the 10,000 cap with ~600 paid** → 4 free lists ≈ **2,350 each** (~9,400) + paid ~600:
- Wave 1: Segment **A ≈ 2,950** (free + paid), Segment **B ≈ 2,350**
- Wave 2: Segment **C ≈ 2,350**, Segment **D ≈ 2,350**
- Finale: Segment **E ≈ 600** (paid)

This reaches ~10,000 people **once** across the month (each in their wave) + the paid finale. To reach MORE than the cap in one month, clear-and-reload between waves (rotation) — same 5 CSVs loaded in phases, never exceeding the cap at once.

> **When asked "what CSVs do I need this month?" → answer: 5 (four free + one paid), sized so `free×4 + paid ≤ contact-cap`, free lists roughly equal. State the count per list and the per-send totals (Wave 1 A = free+paid, etc.).**

## Key Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lerboi/Auto-Email-Campaign](https://github.com/lerboi/Auto-Email-Campaign) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
