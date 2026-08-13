---
trigger: always_on
description: A Houston-focused events aggregator that scrapes venue websites, uses LLM extraction, and presents events on a public calendar with an internal review dashboard.
---

# Group Events — Houston Music & Events App

## Project Overview
A Houston-focused events aggregator that scrapes venue websites, uses LLM extraction, and presents events on a public calendar with an internal review dashboard.

**Live URLs:** houeventlist.com (Houston, live) — atxeventlist.com (Austin, in progress)
**Deployed on:** Railway
**Repo:** Big-Junk:group-events

---

## File Structure
```
group-events/
├── universal_scraper.py      # Main scraper — Selenium + LLM extraction
├── review_dashboard.py       # Flask app — review dashboard + public calendar + API
├── templates/
│   ├── review.html           # Review dashboard UI
│   └── calendar.html         # Public calendar UI
├── Dockerfile                # Web service (Flask)
├── Dockerfile.cron           # Scraper service
├── requirements.txt
└── CLAUDE.md
```

**Local paths:**
- Big-Junk: `/Users/drewcollier/Documents/Work/Xcode/Web Data Dev/AI/group-events/`
- iMac: `/Users/drew/Documents/Python/GRClaude1/`

---

## Tech Stack
- **Backend:** Python 3.11, Flask
- **Scraping:** Selenium + ChromeDriver (headless Chrome)
- **LLM:** OpenAI GPT-4o-mini (default), GPT-4o, Claude Sonnet, Groq/Llama 3.1 70B
- **DB:** PostgreSQL on Railway (psycopg2), SQLite fallback locally
- **Frontend:** Vanilla JS, Jinja2 templates
- **Deployment:** Railway (web service + daily cron + weekly cron)

---

## Database Schema

### events
```sql
id, name, date, doors_time, start_time, venue, location, city, state,
price, ticket_url, description, genre, confidence, notes, status,
created_at, approved_at, event_type, visible
```

**status values:** `pending`, `approved`, `rejected`, `canceled`, `possible_duplicate`
**event_type values:** `music`, `comedy`, `open_mic`, `happy_hour`, `private_event`, `other`
**visible:** boolean, default true for music/comedy, false for others

### past_events
Same schema as events plus `archived_at` timestamp.

### venue_cache
```sql
venue_key, content_hash, last_scraped
```
Used for hash-based change detection to skip LLM when page unchanged.

---

## Scraper Architecture

### Scrape Modes
```bash
python3 universal_scraper.py --mode daily       # Hash check, new events only
python3 universal_scraper.py --mode weekly      # Full scrape + canceled event detection
python3 universal_scraper.py --mode onboard     # New venue, skip hash, filter past events
```

### Additional Flags
```bash
--venue <key>        # Scrape single venue (e.g. --venue white_oak)
--llm <provider>     # gpt4o, gpt4o-mini (default), claude, groq
--dry-run            # Extract and print without saving to DB
--auto-approve       # Auto-approve events on insert (use with onboard)
```

### LLM Providers
- `gpt4o-mini` — default, best cost/quality balance
- `gpt4o` — fallback for complex venues
- `claude` — Claude Sonnet 4, good quality, higher cost
- `groq` — Llama 3.1 70B via Groq, free tier available

### Key Functions
- `scrape_page()` — Selenium page fetch with scroll + pagination support
- `get_content_hash()` / `get_stored_hash()` / `update_stored_hash()` — hash caching
- `pre_filter_dates()` — informational date logging (not used for filtering)
- `extract_events_with_llm_raw()` — main LLM extraction function
- `parse_white_oak_html()` — custom BeautifulSoup parser for White Oak
- `save_to_database()` — insert with exact + partial duplicate detection
- `check_canceled_events()` — weekly mode canceled event flagging

### Duplicate Detection
- **Exact match** (name + date + venue + start_time) → auto-skip
- **URL match** (`ticket_url` and/or `event_url` identifies the same event) → update in place (title/time/openers/price/URL drift all get refreshed on the existing row). Whether each URL field requires `start_date` to also match is controlled per-venue by three toggles stamped onto each event dict in `scrape_venue` — `dupe_use_ticket_url`, `dupe_use_event_url`, `dupe_use_start_date` (all default `True`, i.e. today's safe default: a URL match alone isn't enough, the date must agree too). A venue only gets `dupe_use_start_date: False` after confirming its URL is genuinely unique per occurrence (not a reused comic-profile or generic template page — see Improv Houston and Dan Electro's below) — only then does a URL match with a *different* date get treated as a correction (updates `start_date` in place) rather than falling through to a duplicate insert. If the corrected row was already `approved` and the venue isn't `auto_approve`, the correction also reverts it to `pending` so a human reviews the new date before it's public again.
- **Partial match** (same venue + date, fuzzy name similarity ≥ per-venue `duplicate_threshold`, default 0.5) → insert as `possible_duplicate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drouston/group-events](https://github.com/drouston/group-events) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
