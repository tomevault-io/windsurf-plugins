---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Splunk Quest** — a progress tracker for the team's Splunk certification ramp-up. Flask + SQLite,
server-rendered. No build step, no npm, no test framework, no accounts. All application state is
one SQLite file; the whole deployment is one container.

The curriculum is not invented here. It is extracted from the two `*-learning-path.md` reports at
the repo root, which are the **human source of truth**. Read *The content pipeline* below before
changing any content — it is the part that requires reading several files to see.

## Commands

Everything runs from `app/`. `seed.py`, `check.py` and `refresh_course_urls.py` are stdlib-only;
`app.py` and `smoke.py` need Flask, so a bare `python app.py` outside the venv is the usual
first-run stumble.

```bash
python3 -m venv .venv && .venv/bin/pip install -r app/requirements.txt

cd app
../.venv/bin/python seed.py     # curriculum.json -> ../splunk-quest.db (idempotent)
../.venv/bin/python app.py      # http://localhost:5000 — debug, auto-reload, dev only
```

```bash
cd app
../.venv/bin/python check.py                    # validate curriculum.json; prints a summary
../.venv/bin/python smoke.py                    # every route, against a throwaway database
../.venv/bin/python refresh_course_urls.py --dry-run   # re-scrape Splunk's catalog, report only
../.venv/bin/python refresh_course_urls.py      # ...and rewrite the urls in curriculum.json
```

`check.py` runs inside `seed.py` and refuses to load a file that fails, so seeding a broken
`curriculum.json` is not possible.

**There is no way to run a single test.** `smoke.py` is one `main()` with `ok(label, condition)`
assertions grouped under printed headings (`curriculum`, `identity`, `ticking`, `badges`,
`rollout`, `guards`, `re-seed`, …), no test framework and no selector. It builds its own database
in a tempdir, runs in a couple of seconds, and prints `✓`/`✗` per assertion with a failure list at
the end — run the whole thing. To iterate on one area, add assertions next to the matching heading.

Docker (the recommended deployment; the entrypoint re-seeds on every start):

```bash
docker compose up -d --build && docker compose logs -f
docker compose exec app python check.py
```

Port 5000 is contested — `python app.py` sits there in development and macOS AirPlay Receiver
hides there too. `SPLUNK_QUEST_PORT` moves the container's host port. Full deployment story,
including nginx, TLS and backups, is in `DEPLOY.md`.

## The content pipeline

```
splunk-security-learning-path.md        the reports — human source of truth
splunk-observability-learning-path.md
        │  hand-transcribed, section by section (every record carries `source`)
        ▼
app/curriculum.json                     the machine copy: certs, paths, tiers, items, resources
        │  app/refresh_course_urls.py writes the per-item `url` onto it
        │  app/check.py validates it (slugs, kinds, cert refs, cost tie-outs, links)
        ▼
app/seed.py                             upsert into SQLite — idempotent, re-run at will
        ▼
splunk-quest.db                         content tables read-only at runtime
        ▼
app/app.py + app/db.py + templates/     routes, queries, server-rendered pages
```

`curriculum.json` is **hand-built, not parsed**, and deliberately so: the same course appears in
up to five tables across the reports at different granularities and spellings, cells carry a
`✅ / ❌ / — / ★ / ¹²³` vocabulary, subtotals sit inline, and the free-course inventories are
`·`-delimited prose. A parser would need an alias map and constant maintenance for no benefit at
this size.

**When the plan changes, edit both the `.md` and `curriculum.json`, then re-seed.** The
`update-curriculum` skill (`.claude/skills/update-curriculum/`) walks that end to end — use it for
any content change.

## The invariant everything else rests on: content vs data

`app/schema.sql` splits the tables in two, and `seed.py` treats them differently:

| | tables | what `seed.py` does |
|---|---|---|
| **Content** — from `curriculum.json` | `path`, `tier`, `item`, `cert` | upserts by slug (or `path_id`+`code`); items absent from the JSON are **deleted**, and their progress rows cascade — *except* items in the `team-` slug namespace, which are the team's, not the file's |
| **Content, replaced wholesale** | `lean_stage`, `resource` | `DELETE` then re-insert on every seed |
| **Data — written by people** | `user`, `enrollment`, `progress`, `link`, `milestone`, `milestone_person`, `item_edit` | **never touched** |

That is why "deploy" is just `git pull && docker compose up -d --build`: there is no migration
step, and re-seeding cannot take the team's progress with it. The
`N people, N progress marks and N team milestones preserved` line `seed.py` prints is the receipt.
`smoke.py` asserts it. **Do not add a write to a data table from `seed.py`.**

**The rollout is entirely data.** A milestone is three things — which certifications, who needs
them, by when — written by the team at `/rollout` and stored in `milestone` + `milestone_person`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sispehar/splunk-quest](https://github.com/sispehar/splunk-quest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
