---
trigger: always_on
description: handles most pointer-tracking libraries).
---

<!-- Copy everything below this line into your agent's system prompt to teach
     it how to use the Periscope MCP tools effectively. -->

# Website & Web-App Testing with Periscope

You have access to Periscope, an MCP server exposing 74 Playwright/Chrome tools
for testing websites and web apps (static sites, SPAs, and apps behind a login).
Call `describe_tools(category?)` anytime for the full catalog with parameters
and workflows.

## Core model

- **Sessions are your main workflow.** `open_session(url)` returns a
  `session_id` and keeps a real browser page alive across tool calls — state,
  cookies, console output, and network traffic accumulate. Multi-step work
  (login, forms, SPA flows, debugging) belongs in a session. One-shot tools
  that take a bare `url` open and close a throwaway page each call; use them
  only for single lookups.
- **Isolation:** calls without a `project` run in private, isolated browser
  contexts — no cookies or login state are shared between them. Pass
  `project` when you *want* shared state (authenticated testing).
- Sessions expire after 300s idle and there is a 20-session cap (both
  env-overridable). Close sessions with `close_session` when done. A
  "session not found" error names *why* it's gone — idle-expired, evicted at
  the cap, or the browser crashed/restarted (that one also drops login state,
  so re-run `login_project`). Reopen with `open_session` and continue; don't
  retry the dead id.
- Every tool returns JSON. Failures come back as `{"success": false, "error":
  ...}` — read the error, they are written to tell you the fix (wrong selector,
  expired session, missing argument).

## Standard workflows

**Explore then act.** Orient with ONE call: `get_page_map(session_id)` —
every interactive element and landmark with its role, accessible name, state,
and a ready-to-use selector, in document order (interactive elements with no
accessible name come back flagged `unnamed`: an accessibility finding).
For targeted lookups, `get_page_elements(session_id, "button")` lists matches
for a selector and `find_element(session_id, text="Submit")` finds the best
selector by text. Never guess selectors.

**Static audit of a site:**
`create_project(name, base_url)` → `test_project(project)` (crawls and runs
visual/accessibility/functionality/seo/performance/geo checks on every page,
saves a JSON report) → `get_report(path)`. For one page: `test_url(url)`.
The `geo` check covers AI/agentic-search readiness: robots.txt access for AI
crawlers, llms.txt compliance, WebMCP form annotations, and JSON-LD presence.

To capture a site (not just audit it): `crawl_project(project, meta=true)` adds
each page's title + description, and `save_md=true` saves every crawled page as
readable Markdown to `data/fetches/<project>/` — captured during the crawl on the
loaded page, so behind-login and JS-rendered pages work.

The crawl is **deterministic and sitemap-seeded**: it seeds from
`sitemap.xml`/robots.txt when present and sorts links before applying the cap,
so the *same* site gives the *same* page subset every run — before/after
re-tests are comparable (a page can't silently drop out and look "fixed").
`max_pages` caps how many pages are tested (default 20); **`max_pages=0` tests
the whole site** (unbounded, stops at a 2000-page safety ceiling and flags
`ceiling_hit`). Whatever the cap leaves out comes back in `pages_not_tested[]`
(≤100, else a count) — coverage is never silent. `test_project` also returns a
`coverage` delta (`pages_added`/`pages_dropped`) vs the previous report. Set
`use_sitemap=false` for a pure link-crawl.

**Authenticated testing:** configure once with `set_form_login` /
`set_basic_auth` / `set_cookies`, then `login_project(project)`. Pass
`project` to `open_session` so the session shares the logged-in context.
`copy_auth(from, to)` moves auth between projects on the same domain.
Auth can expire mid-run (token rotation, short sessions) — Periscope detects
it rather than masking it: `test_project`/`crawl_project` preflight the auth
and re-login automatically (see `auth_check` in the response), pages that land
on the login page come back as `status: "auth_lost"` with an error issue (never
plain success), and `open_session` warns when it lands on the login page. On
any of those signals, run `login_project` again.

**Logins you can't automate** (2FA/MFA, SSO/OAuth redirects, CAPTCHA, magic
links): call `interactive_login(project)` — it opens a **visible** browser
window (requires a display on the server); the user logs in by hand, then you
call `save_login(project)` to capture the session. The project then opens
authenticated sessions **headlessly** with the saved login. To simply watch or
hand-drive any session, use `open_session(url, headed=true)`.

**Multi-step flows:** batch steps into one `interact_and_test` call instead of
many single calls — it supports 25 actions (click, fill, select, wait_for,
wait_for_text, wait_for_network, navigate, hover, press_key, upload_file,
evaluate_js, drag, scroll…), takes a screenshot at the end, and can run checks
via `run_checks`. Use `continue_on_error=true` when later steps make sense
even if one fails.

**Form testing:** `auto_fill_form(session_id)` detects fields, infers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [segentic-lab/periscope-mcp](https://github.com/segentic-lab/periscope-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
