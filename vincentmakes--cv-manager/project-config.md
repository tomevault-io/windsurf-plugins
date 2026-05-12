---
trigger: always_on
description: CV Manager is a self-hosted, Docker-deployed CV/resume management system. It runs two Express servers: an admin interface (port 3000) for editing and a read-only public site (port 3001) for visitors.
---

# CLAUDE.md — AI-Assisted Development Guide

## Project Overview

CV Manager is a self-hosted, Docker-deployed CV/resume management system. It runs two Express servers: an admin interface (port 3000) for editing and a read-only public site (port 3001) for visitors.

**Tech stack:** Node.js 20, Express, SQLite (better-sqlite3), vanilla JS frontend (no frameworks), plain CSS.

## Quick Reference

```bash
npm start              # Production (admin :3000, public :3001)
npm run dev            # Development with --watch
npm test               # Run all tests
npm run test:backend   # Backend tests only
npm run test:frontend  # Frontend tests only
```

There is no linter or formatter configured. Tests use Node's built-in `node:test` module.

## Project Structure

```
src/server.js              # Express server (admin + public), DB schema, all API routes
public/index.html          # Admin interface HTML template
public/shared/admin.js     # Admin UI logic (modals, CRUD, datasets, sections)
public/shared/scripts.js   # Shared frontend utilities (API calls, date formatting, rendering)
public/shared/styles.css   # All CSS (admin toolbar hidden on public via .no-print)
public/shared/i18n.js      # Internationalization module
public/shared/i18n/*.json  # Translation files (en, de, fr, nl, es, it, pt, zh)
public-readonly/index.html # Public-facing HTML template (read-only)
tests/backend.test.js      # Backend API tests (spawns server on random port)
tests/frontend.test.js     # Frontend unit tests
data/                      # SQLite database (created at runtime)
version.json               # Current version (used by update checker)
```

## Architecture

### Dual-Server Model

- **Admin server** (PORT=3000): Full CRUD, file uploads, settings. Serves `public/`.
- **Public server** (PUBLIC_PORT=3001): GET-only, rate-limited (60 req/min), security headers, filters sensitive data (email/phone/IDs). Serves `public-readonly/`.
- **Public-only mode**: Auto-detected when filesystem is read-only, or forced via `PUBLIC_ONLY=true`.

### Database

SQLite with WAL mode. Key tables: `profile`, `experiences`, `certifications`, `education`, `skill_categories`, `skills`, `projects`, `section_visibility`, `saved_datasets`, `custom_sections`, `custom_section_items`, `settings`.

Automatic migrations run on startup — new columns are added dynamically, legacy date formats normalized.

### Dataset System

Multiple CV snapshots can be saved. One is marked "default" and served at `/` on the public site, decoupling live edits from the public page. Other datasets can be made public at `/v/{slug}`.

## Versioning System

The project follows [Semantic Versioning](https://semver.org/):

- **MAJOR** (X.0.0): Breaking changes or major architectural shifts
- **MINOR** (x.Y.0): New features (new sections, new settings, new capabilities)
- **PATCH** (x.y.Z): Bug fixes, translation fixes, small improvements

**Version must be updated in three files:**
1. `package.json` → `"version"` field
2. `version.json` → `"version"` field
3. `package-lock.json` → `"version"` field (appears twice: top-level and inside `packages[""]`)

All three must always match. The admin UI checks `version.json` on GitHub to show update notifications.

**Changelog:** `CHANGELOG.md` follows [Keep a Changelog](https://keepachangelog.com/) format. Every version bump must have a corresponding changelog entry with the date in `YYYY-MM-DD` format. Use categories: `Added`, `Changed`, `Fixed`, `Removed`.

### MANDATORY: Version Bump Checklist

**Every code change that is committed MUST include a version bump.** This is not optional — skipping it breaks the update notification system and makes releases untrackable.

**Exception — documentation-only and test/CI-only changes do NOT get a version bump.** Changes to `docs/`, `mkdocs.yml`, `requirements-docs.txt`, `USER_GUIDE.md`, `TRANSLATING.md`, `README.md`, `CHANGELOG.md`, `tests/`, or `.github/workflows/` (without an accompanying app change) must **not** bump `package.json` or `version.json`. The version number is used by the admin UI to detect app updates — bumping it for docs-only or test/CI-only changes would create false update notifications for users.

Before committing, always complete these steps:

1. **Determine bump type** — Bug fix → PATCH, new feature → MINOR, breaking change → MAJOR
2. **Update `package.json`** — Increment the `"version"` field
3. **Update `package-lock.json`** — Increment the `"version"` field in both the top-level object and inside `packages[""]`
4. **Update `version.json`** — Increment the `"version"` field (must match `package.json`)
5. **Update `CHANGELOG.md`** — Add a new `## [x.y.z] - YYYY-MM-DD` entry at the top with a description of the change under the appropriate category (`Added`, `Changed`, `Fixed`, `Removed`)

**Do not** batch multiple unrelated changes into a single version bump — each logical change set gets its own version.

## Internationalization (i18n)

### How It Works


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentmakes/cv-manager](https://github.com/vincentmakes/cv-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
