---
trigger: always_on
description: Headful job-board scraper. Scrapes LinkedIn (login-gated) and Wuzzuf
---

# AGENTS.md — RTJobs

## What this is
Headful job-board scraper. Scrapes LinkedIn (login-gated) and Wuzzuf
(Cloudflare-gated) via [scrapling](https://scrapling.readthedocs.io) stealth
browser sessions, persists jobs to SQLite, posts new jobs to one Telegram
channel and failure alerts to another. Scheduled in Docker via ofelia
(every 6 min). Chrome runs headful under Xvfb with CDP on port 9222 for
live debugging / manual 2FA solves.

## Commands
```bash
source .venv/bin/activate.fish      # shell is fish; venv is Python 3.14
python main.py                      # run all enabled boards
python main.py --reset-login        # clear LinkedIn retry/cooldown state
python -m py_compile <files...>     # no linter/typechecker configured — compile check + offline tests are the verification loop
docker compose up -d --build        # scheduled container run (ofelia)
docker compose logs -f scraper
```
There is NO test framework. Verification = ad-hoc offline scripts that run
extraction/parsing functions against the fixture files in `markup/` (see
"Offline testing" below).

## Architecture
```
main.py                  orchestrator; BOARDS list; --reset-login
config.py                ALL env config (single source of truth)
core/db.py               SQLite: jobs, seen_ids, runs, login_state
core/telegram.py         notify_jobs (jobs channel) / notify_failure (alert channel)
core/markup.py           sanitized HTML snapshots -> markup/<site>/snapshots/<kind>/
core/login_state.py      LinkedIn retry counter + escalating cooldown (5m/15m/30m)
core/browser.py          patch_no_load_wait — see Gotchas #1
core/human.py            random human-like delays
boards/base.py           JobBoard ABC + load_board_selectors()
boards/linkedin/         login.py (state machine) + scraper.py (Spider)
boards/wuzzuf/           scraper.py (Spider, solve_cloudflare=True)
boards/indeed/           scraper.py (Spider, solve_cloudflare=True) — see INDEED.md
markup/<site>/selectors.json   ALL CSS selectors live here, never in code
```
Job dict shape everywhere: `source, external_id, title, company, posted_at,
description, link, extra(dict), scraped_at`.

## Gotchas (hard-won — read before touching browser code)
1. **scrapling waits for the browser `load` event** on every navigation
   (`page.goto(wait_until="load")` default + `_wait_for_page_stability`).
   LinkedIn/Wuzzuf never fire `load` (hanging trackers) → every fetch times
   out. Fix: `core/browser.py:patch_no_load_wait` is passed as `page_setup`
   to every session. CONTRACT: scrapling's **async** sessions do
   `await params.page_setup(page)` and `await page.goto(...)` — the patch
   detects async pages via `inspect.iscoroutinefunction(page.goto)` and must
   return a coroutine + install `async def` wrappers. Sync sessions get sync
   wrappers and `None`. Verified in
   `.venv/lib/python3.14/site-packages/scrapling/engines/_browsers/_stealth.py`.
2. **Chrome lifecycle + CDP attach**: WE launch Chrome
   (`core/browser.py:launch_cdp_chrome`) with `--remote-debugging-port` —
   never let scrapling launch it: playwright forces
   `--remote-debugging-pipe`, which DISABLES the HTTP DevTools endpoint
   (localhost:9222 would serve nothing). Boards pass `cdp_url=` to every
   session, and `install_cdp_default_context_patch()` replaces the cdp
   branch of scrapling's `start()` to reuse `browser.contexts[0]` —
   scrapling's own path calls `new_context()`, which is isolated from the
   profile's cookies (would silently drop the LinkedIn session / Wuzzuf
   cf_clearance every run). CAREFUL: after a `new_context()` call the
   contexts list is reordered and index 0 becomes the ISOLATED one — grab
   contexts[0] straight after `connect_over_cdp`. The container needs
   `network_mode: host` because Chrome binds CDP to loopback and
   docker-proxy can't forward to a container-loopback listener.
3. **Wuzzuf data comes from the SSR blob**, not just the DOM:
   `window.Wuzzuf.initialStoreState.job.collection` (full entities: HTML
   description/requirements, exact `postedAt` `MM/DD/YYYY HH:MM:SS`,
   salary, career level…). It is parsed from `page.content()` with marker
   regex + brace balancing (`_extract_state`) — NOT `page.evaluate`
   (evaluate silently failed under stealth isolated contexts).
4. **Wuzzuf pagination** is a page index: `?q=&start=0`, `start=1`, …
   (15 jobs/page). Stop condition: first `external_id` already in
   `seen_ids` (default sort is by date). Job id = numeric prefix of the
   slug: `/jobs/p/<id>-<slug>`.
5. **LinkedIn login**: fetch `https://www.linkedin.com/login`, fill with
   human-like typing (EN+AR aware), then `_verify_routing` waits
   event-based via `page.wait_for_url` for `(feed|/jobs|checkpoint|security_verification)`
   — do NOT put `login` in that pattern (current URL matches it instantly).
   Logged-in detection (`_FEED_OK`) is `/feed` ONLY — guests get redirected
   to `/jobs` too, so matching `/jobs` treats a guest session as logged in
   (this bug shipped once). Checkpoint/2FA → alert + pause up to
   `CHECKPOINT_WAIT_SECONDS` for a manual solve over CDP. Profile wipe
   (after max retries + cooldown) must happen BEFORE the browser starts
   (`LinkedInBoard.run`), never inside a page_action of a live Chrome. Same

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [y2youssef/RTJobs](https://github.com/y2youssef/RTJobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
