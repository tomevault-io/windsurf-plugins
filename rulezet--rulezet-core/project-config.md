---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**Rulezet** is an open-source community platform for sharing, evaluating, and managing cybersecurity detection rules (YARA, Sigma, Suricata, Zeek, CRS, Nova, NSE, Wazuh, Elastic). It is a Flask + Vue.js 3 application backed by PostgreSQL. Live at [rulezet.org](https://rulezet.org/).

---

## Commands

### Run the app (development)
```bash
source env/bin/activate
./launch.sh -l       # or: FLASKENV=development python3 app.py
```

### Initialize the database (first time)
```bash
python3 app.py -i    # creates tables + admin user, prints credentials
```

### Recreate the database (drop + reinit)
```bash
python3 app.py -r
```

### Run all tests
```bash
./launch.sh -t       # or: FLASKENV=testing pytest tests
```

### Run a single test file
```bash
FLASKENV=testing pytest tests/rules/test_rule.py
FLASKENV=testing pytest tests/rules/test_search_rules.py -k "test_name"
```

### Database migrations
```bash
flask db migrate -m "description"
flask db upgrade
```

### Gunicorn (production)
```bash
gunicorn -w 4 wsgi:app
```

### Backup / restore
```bash
./backup/scripts/backup_rulezet.sh
./backup/scripts/restore_rulezet.sh
```

---

## Configuration

`config.py` defines three environments selected via `FLASKENV`:

| `FLASKENV`    | DB                             | Notes                        |
|---------------|--------------------------------|------------------------------|
| `development` | `postgresql:///rulezet`        | `DEBUG=True`, sessions in PG |
| `testing`     | `sqlite:///rulezet-test.sqlite`| CSRF disabled, sessions in FS|
| `production`  | `postgresql:///rulezet`        | `DEBUG=False`                |

Secrets live in `.env` (`SECRET_KEY`, `MAIL_PASSWORD`). The app runs on `127.0.0.1:7009` by default.

---

## Architecture

### Entry points

| File | Role |
|------|------|
| `app.py` | CLI entry point — parses `-i/-r/-d` flags, starts Flask dev server |
| `wsgi.py` | Gunicorn entry point |
| `app/__init__.py → create_app()` | Flask application factory; registers blueprints, extensions, starts background worker |

### Blueprints (UI layer — `app/features/`)

| Blueprint | URL prefix | Module |
|-----------|-----------|--------|
| `home_blueprint` | `/` | `app/home.py` |
| `account_blueprint` | `/account` | `app/features/account/account.py` |
| `rule_blueprint` | `/rule` | `app/features/rule/rule.py` |
| `bundle_blueprint` | `/bundle` | `app/features/bundle/bundle.py` |
| `tags_blueprint` | `/tags` | `app/features/tags/tags.py` |
| `jobs_blueprint` | `/jobs` | `app/features/jobs/jobs.py` |
| `api_blueprint` | `/api` | `app/api/api.py` (Flask-RESTX, CSRF exempt) |

### REST API (Flask-RESTX — `app/api/`)

Swagger UI is accessible at `/api/`. Namespaces:

| Path | Module | Auth |
|------|--------|------|
| `/api/rule/public` | `app/api/rule/rule_public_api.py` | None |
| `/api/rule/private` | `app/api/rule/rule_private_api.py` | `X-API-KEY` header |
| `/api/bundle/public` | `app/api/bundle/bundle_public_api.py` | None |
| `/api/bundle/private` | `app/api/bundle/bundle_private_api.py` | `X-API-KEY` header |
| `/api/account/public` | `app/api/account/account_public_api.py` | None |
| `/api/account/private` | `app/api/account/account_private_api.py` | `X-API-KEY` header |

API key auth is enforced via `@api_required` from `app/core/utils/decorators.py`, which calls `verif_api_key()` in `app/core/utils/utils.py`. The key is passed in the `X-API-KEY` request header.

### Data model (`app/core/db_class/db.py`)

All SQLAlchemy models live in one file. Key models:

| Model | Description |
|-------|-------------|
| `User` | Auth + profile; has `api_key`, `admin`, `is_verified`, gamification backref |
| `Rule` | Core entity: `format`, `title`, `to_string` (raw content), `uuid`, `source`, `github_path` |
| `FormatRule` | Registry of supported rule formats |
| `Bundle` | Named collection of rules (many-to-many via `BundleRuleAssociation`) |
| `BundleNode` | Tree node for bundle's file-explorer view (`folder` or `file`, recursive self-ref) |
| `Tag` | Taxonomy tags with color/icon/galaxy metadata; linked to rules and bundles via association tables |
| `RuleEditProposal` | PR-style edit request with `status` (pending/approved/rejected) |
| `Comment` / `CommentBundle` | Comments on rules and bundles |
| `RuleVote` / `BundleVote` | Per-user up/down votes |
| `RuleFavoriteUser` | User favorites |
| `InvalidRuleModel` | Rules that failed validation on import |
| `BackgroundJob` + `BackgroundJobLog` | Persistent job queue for long-running tasks |
| `Gamification` | Per-user contribution points and level; auto-updated via SQLAlchemy `before_flush` event listener `receive_before_flush()` |
| `RuleSimilarity` / `SimilarResult` | Fuzzy similarity scores between rules (TF-IDF + FAISS + rapidfuzz) |
| `ImporterResult` / `UpdateResult` / `RuleStatus` / `NewRule` | History tracking for GitHub imports and rule update scans |

### Business logic layer (`app/features/*/` and `app/core/`)

Each feature has a `*_core.py` file with pure Python DB logic, called by both blueprints and API namespaces:

| File | Key functions |
|------|--------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rulezet/rulezet-core](https://github.com/rulezet/rulezet-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
