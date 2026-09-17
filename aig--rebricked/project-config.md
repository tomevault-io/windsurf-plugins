---
trigger: always_on
description: Guidance for AI agents (and humans) working in the **rebricked** repo.
---

# AGENTS.md

Guidance for AI agents (and humans) working in the **rebricked** repo.

## What this project is

A single static page that answers one question: *"What happened to the thing Databricks
used to call X?"* It lists Databricks product/feature **renames and deprecations** -
sourced, dated, searchable - dressed as the Databricks console. There is no build step,
no framework, no backend. (rebricked = **re**named or de**pre**cated.)

## The one rule

**Real, sourced changes only. Never be confidently wrong.**

There is no `kind` field: **`status` is the sole discriminator**, and it stores only what
can't be calculated. `status` is a `{ value, link, date }` object: **`value`** holds the
discriminator (the states listed below), `link` is the official doc backing that call, and
`date` is the day it was confirmed. Its values (i.e. `status.value`):
- **`active`** - any name in use now. This covers BOTH a genuinely new capability AND the
  current name of something that was renamed. The two are *not* stored separately (that would
  be redundant) - they're **calculated**: a standalone feature carries its own `introducedAt`;
  the current tip of a rename chain carries `from` and has a `renamed` card pointing at it.
- **`renamed`** - a superseded former name (needs a `to` date and a `successorId` at the name
  that replaced it). A rename is therefore one-or-more `renamed` cards chained to an `active`
  current name.
- **`deprecated` / `legacy` / `retired`** - retired or replaced: a *different* thing took over
  (or nothing did), usually with a different API/format. That "different thing" call is a human
  judgement (the opposite of a rename), so it is stored, not derived.

The UI groups `active` names (features and current rename tips alike) under the **Active**
filter. Release maturity (Preview vs GA) is a separate, orthogonal `releases` timeline (an
ordered `{type, date}` array; last stage = current), so a card can be e.g. `active` but
currently `public-preview`, or `legacy` but `beta`.

Every entry needs an official source from its own vendor (Databricks or Microsoft Learn docs
for a Databricks entry; `docs.snowflake.com` or the Snowflake blog for a Snowflake one) and a `verified`
date. If you cannot verify a claim against a live doc, do not add it - flag it instead.
See [CONTRIBUTING.md](CONTRIBUTING.md) for the field rules, and
[`docs/`](docs/) for the full documentation set: [tutorials](docs/tutorials/) to learn the repo,
[how-to guides](docs/how-to/) per task, [reference](docs/reference/) to look a field or flag up,
[explanation](docs/explanation/) for why any of it is shaped this way.

When asked to "validate" the list, that means fact-check each entry against its cited
source and current vendor naming - not just run the schema check.

## Vendors

The repo tracks more than one vendor. `kb/<vendor>/` is a vendor; `kb/posts/` is not (it is the
guides). Everything about an entry is identical across vendors - the same `status` discriminator,
the same required fields, the same "real, sourced changes only" rule - and only four things are
per-vendor:

- **The source of truth for claims.** Databricks entries cite Databricks or Microsoft Learn docs;
  Snowflake entries cite `docs.snowflake.com` or the Snowflake blog. Never cross them.
- **`VALID_CATEGORIES`** in [`validate.py`](scripts/validate.py) is a dict keyed by vendor, because
  the categories mirror each vendor's own console vocabulary.
- **The console chrome.** Each vendor's generated pages wear that vendor's own console - the
  homage only works if it names the right one. [`scripts/chrome.py`](scripts/chrome.py) owns
  every rail: Databricks' is parsed out of `app.js`'s `NAV` (so the static pages can never drift
  from the SPA), Snowflake's is Snowsight's, declared there. `<html data-vendor="...">` then
  swaps the palette in [`styles.css`](www/styles.css). A vendor with no rail there renders in
  the site's own chrome.
- **The rail-coverage check.** "Every entry is reachable from a section" now runs for *every*
  vendor that has a rail in `chrome.py`, in both directions. A vendor with no rail is skipped and
  reaches readers through its generated hub at `/{vendor}/`.
- **The hub layout.** `/databricks/` is a document, because the Databricks console content area
  is; `/snowflake/` is Snowsight's home screen - search box, quick actions, one filterable table.
  Every entry link is in the markup either way, so both are equally crawlable.
- **The URL namespace.** `/{vendor}/{id}/`, from `build_entries.py`.

Two things are **not** per-vendor and will fail the gate if you assume otherwise: **entry ids are
globally unique** (the generated pages index every entry by id in one map), and **a rename chain
cannot cross vendors** (`successorId` must point at the same vendor).

The single-page app still fetches only `databricks.features.json`, so `/snowflake/` today is the
generated hub and entry pages, not an app experience. `build_entries.py` knows this and links
Snowflake entry pages to their hub rather than to a `?id=` deep link the app could not serve.

**Adding or editing an entry? Use the [`add-databricks-entry`](agents/add-databricks-entry.md)
skill** (or [`add-snowflake-entry`](agents/add-snowflake-entry.md) for a Snowflake one, which

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aig/rebricked](https://github.com/aig/rebricked) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
