---
trigger: always_on
description: Context for anyone (or any AI) working **on** this plugin. It is a
---

# YouTube Comments AI — plugin guide (for humans & AI agents)

Context for anyone (or any AI) working **on** this plugin. It is a
**self-provisioning** PyRunner plugin (SDK `core.plugins.api`): one form
**Save** creates and keeps in sync a managed Script, its secrets, a managed
Postgres **Database** (the first DB-native example plugin), a state DataStore
and a daily Schedule. It is the reference for a **database-backed** plugin and
consumes three platform seams: Databases (`DatabaseAPI`/`pyrunner_db`), AI
Providers (`pyrunner_ai`) and instance email (`pyrunner_notify.email(html=…)`).
Stage 3 adds **reply automation**: a Google OAuth connect flow (`oauth.py`),
a Reply Brain, per-tag reply policies with hard auto-publish guardrails, a
Reply Queue, and optional spam moderation. Stage 4 adds the **weekly insights
email** (AI-clustered questions → FAQ/content ideas + per-video sentiment
trend). Stage 5 adds **publish-ready testimonials**: a focused AI grading pass
(feature/solid/generic + why) and **avatar archiving** to the instance's
object storage (`pyrunner_storage`/`StorageAPI`, SDK 2.5).

Read this before editing — most bugs you can introduce here are silent
cross-layer desyncs, not syntax errors.

## Architecture: two execution contexts

| Runs in the **web process** | Runs in the **environment venv** (a Script) |
|---|---|
| `apps.py`, `urls.py`, `views.py`, `forms.py`, `templates/` | `worker_body.py` (the managed analyzer script) |
| `provisioning.py` — all SDK calls + YouTube channel resolution at Save | — |
| `oauth.py` — Google consent flow, token exchange/refresh, immediate posting from the Reply Queue | (the worker has its own minimal copies — it can't import web modules) |

The web layer talks to PyRunner **only** through `core.plugins.api`; the
dashboard reads the plugin's own tables via `DatabaseAPI(OWNER).dsn(DB_KEY)`
+ psycopg (`provisioning.db_rows`). The worker is a standalone script that
imports only stdlib + `requests` + injected helpers (`pyrunner_db`,
`pyrunner_datastore`, `pyrunner_ai`, `pyrunner_notify`).

## Contracts you must not break

1. **Secret/config names are a cross-process contract by convention.**
   `forms.SECRET_FIELDS` env keys + `provisioning.OAUTH_TOKEN_KEY` +
   `provisioning.CONFIG_KEYS` + `provisioning.BRAIN_KEYS` must all appear
   literally in `worker_body.py`; `provisioning._worker_code()` fails **loudly
   at Save** if one goes missing. Rename on both sides or not at all.
2. **Import-light / SDK-only.** `views.py`/`provisioning.py` never
   `import core.models|services|tasks`. psycopg is imported lazily inside
   functions.
3. **No `models.py`, no `migrations/`.** SQL DDL lives ONLY in the worker's
   `SCHEMA_SQL` (`CREATE TABLE IF NOT EXISTS`, additive). The web layer must
   tolerate missing tables (`views._dashboard_data` maps `UndefinedTable` →
   the "worker hasn't run" empty state).
4. **Postgres is REQUIRED.** `provision()` fails closed with a friendly
   message when `DatabaseAPI.is_available()` is False. Don't add a DataStore
   fallback — a locked product decision (2026-07-15).
5. **Ownership + idempotency.** Everything owned by slug `yt_comments`; SDK
   upserts key on `(owner_plugin, owner_key)`; re-running `provision()` must
   never create duplicates (covered by `ProvisionTests`).
6. **Secrets are write-only in the form.** Blank = keep existing.

## Engine invariants (the bug-prone parts)

- **The watermark is the correctness core.** One channel-wide sweep
  (`commentThreads.list` + `allThreadsRelatedToChannelId`, `order=time`,
  newest first) pages down to `iso_floor(watermark, start_date)`; the
  watermark **only advances after a COMPLETE sweep** (floor reached or history
  exhausted). Quota/network abort mid-sweep ⇒ watermark stays ⇒ next run
  re-fetches (upserts dedupe). A page-cap stop counts as complete (bounded
  backfill, loudly logged). Persisting is page-by-page with per-page commits.
- **Upserts never clobber analysis.** `UPSERT_COMMENT_SQL` refreshes
  text/likes/`avatar_url` only (fetch-data); `status`/`tags`/`sentiment`/
  `testimonial_grade` are written exclusively by the AI passes.
- **A classification failure never mislabels.** Parse failure / model skip /
  over-cap ⇒ the row stays `pending_analysis` and is retried next run. There
  is NO default tag (the prototype's "default to positive" bug).
- **Grading follows the same rule: ungraded ≠ generic.** A failed grading
  batch leaves `testimonial_grade` NULL and is retried — grading must never
  quietly downgrade a testimonial. Grades are a closed set
  (`TESTIMONIAL_GRADES`: feature/solid/generic), shared by string with
  `views._GRADE_BADGE`.
- **Avatars: stable keys, testimonial authors only, fail-soft.** The storage
  key is ALWAYS `avatars/<channel_id>.jpg` (overwritten on change — published
  hot-links must never break; the stored Content-Type wins over the
  extension). `comments.avatar_url` semantics: NULL = never captured
  (backfill target, `comments.list` 1 unit/50 ids), '' = unavailable — don't
  collapse the two or the backfill loops forever. The `authors` row is only
  written on a successful sync (that's the retry mechanism). Any avatar/
  storage failure is a log line, never a failed run.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hassancs91/PyRunner](https://github.com/hassancs91/PyRunner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
