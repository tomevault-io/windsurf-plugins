---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

Django 6.0 (Python 3.14+), managed with **uv** (not pip/poetry — always `uv sync` / `uv run ...`), Tailwind CSS v4 via CLI, deployed to Vercel (WSGI). Project package is `FlexForge/`; apps live under `apps/`. Database and media storage are both Supabase (Postgres + Storage) in production; SQLite is used automatically in development/tests.

## Commands

- Install deps: `uv sync`
- Dev server: `uv run python manage.py runserver`
- Tests: `uv run python manage.py test` (e.g. `uv run python manage.py test apps.blog` for one app) — this is what CI runs and is the canonical command. The README also mentions `uv run pytest`; pytest-django is configured and works, but CI does not use it.
- Django check: `uv run python manage.py check`
- Tailwind build: `npx @tailwindcss/cli -i ./static/css/input.css -o ./staticfiles/css/global-wvbpenzt.css --minify` (add `--watch` for dev). There is no `collectstatic` step anywhere in this project's pipeline (not in CI, not in the Vercel build) — the built `staticfiles/` output (images, fonts, icons, compiled CSS, JS) is committed directly and served as-is. This is why `STORAGES["staticfiles"]` uses WhiteNoise's plain `CompressedStaticFilesStorage`, **not** `CompressedManifestStaticFilesStorage`: none of these pre-built assets live under any `STATICFILES_DIRS` source `collectstatic` could discover (they're placed directly under `STATIC_ROOT`), so a manifest can never be generated for them — `ManifestStaticFilesStorage`'s strict lookup would 500 on every `{% static %}` tag referencing one (this was a real, live bug on the `main` branch — every page extending `templates/base_seo.html` 500'd via its favicon links). If you add a genuinely new static asset with a `{% static %}` reference, just drop the file under `staticfiles/` directly with its final name (matching the hand-picked cache-busted filename convention below) — don't reach for `collectstatic`.

## Architecture: Django ORM (Supabase-backed)

Content (bio, experience, projects, blog posts, education, awards, applications, hiring/open-to-work status, privacy policy) lives as real Django models — `apps/about/models.py`, `apps/blog/models.py`, `apps/projects/models.py`, `apps/openhire/models.py`, `apps/core/models.py` (`PrivacyPolicy`) — backed by Postgres (Supabase in prod, SQLite in dev/tests). This replaced an earlier "Individual File System" (dataclasses in per-app `data/` files, no DB) — that whole layer (`apps/*/data/`, `apps/core/dynamic_loader.py`) was migrated and removed; don't recreate it.

- **Access pattern preserved on purpose**: `apps/core/data_service.py`'s `DataService`, `apps/about/manager.py`'s `AboutManager`, `apps/core/content_manager.py`'s `ContentManager`, and `apps/openhire/manager.py`'s `OpenHireManager` are the only things views/templates/`apps/seo/schema.py` talk to. They query the ORM internally but hand back the exact same plain `dict`/`list[dict]` shapes the old dataclass files produced (including derived fields like `image_url`/`img_name`/`image_count` on blog/project dicts, built by `apps/core/content_manager.py`'s `_add_image_compat_fields`). Extend these managers rather than querying models directly from views, and preserve the dict shape when you do.
- **Singletons** (`Profile`, `HiringProfile`, `OpenToWorkProfile`, `PrivacyPolicy`) extend `apps/core/models.py`'s `SingletonModel` (forces `pk=1`, blocks delete) and use `.load()` (get-or-create) rather than a plain query.
- **Slugs are computed, not stored data**: `BlogPost.slug`/`Project.slug` are `slugify(title)`, resolved via `apps/core/base_views.py`'s `DetailView.get_item_by_slug(queryset, slug, to_dict_fn)` — an indexed DB lookup (`get_object_or_404`), not a linear scan.
- **Ordered M2M**: `Profile.skills_highlight` goes through `ProfileSkillHighlight` (profile, skill, `order`) because the sequence is editorial — it becomes the JSON-LD `knowsAbout` array. Read it via `profile.skill_highlights` (prefetch `skill_highlights__skill`), never the bare M2M, which returns `Skill.Meta.ordering` (pk) order. A through model rules out `filter_horizontal` (`admin.E013`), so the admin uses a `TabularInline` with `autocomplete_fields`. `Project.tech_stack` stays a plain M2M with `filter_horizontal` — its order genuinely doesn't matter. Note Django cannot `AlterField` a M2M's `through=`; changing one needs `RemoveField` + `AddField` with the data copied out first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ridwaanhall/ridwaanhall-com](https://github.com/ridwaanhall/ridwaanhall-com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
