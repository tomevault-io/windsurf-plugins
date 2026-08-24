---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cafe-Grader is an online programming contest and assignment grading platform (used at Chulalongkorn University). Students submit code, which is automatically compiled and evaluated against test cases. Instructors manage problems, contests, groups, and view reports.

**Backlog of deferred work** lives in `doc/backlog.md` — design refactors, "someday" follow-ups, and decisions parked for a longer session. Check it when starting work in an area that might already have a planned change; add to it (don't bury in comments) when you defer something non-trivial. Inline `# TODO(scope): …` is fine for one-liners anchored to code; GitHub Issues are for scheduled/assigned work.

## Version Control

Local VCS is **Mercurial (hg)**, mirrored to **GitHub** via the **hg-git** extension. Use `hg` for all local operations (`hg status`, `hg commit`, `hg diff`, `hg push`). Issues and PRs live on GitHub — use the `gh` CLI for those (e.g., `gh issue close 51`).

## Tech Stack

- **Ruby 3.4.4, Rails 8.0.0** (with `load_defaults 7.0`)
- **MySQL 8.0+ only** (Oracle MySQL or Percona; **MariaDB unsupported** — every table uses the `utf8mb4_0900_ai_ci` collation, which MariaDB lacks; enforced by `test/schema_collation_test.rb`, rationale in `doc/decisions.md`). Primary DB: `grader`, queue DB: `grader_queue`
- **Propshaft** asset pipeline, **ImportMap** for JS, **dartsass-rails** for CSS (no Node/yarn dependency)
- **Hotwire** (Turbo + Stimulus), jQuery (legacy), Bootstrap 5
- **HAML** templates
- **Solid Queue** for background jobs, **Solid Cache**, **Solid Cable**
- **Puma** web server with Thruster

## Development Commands

```bash
# Start all dev processes (server, CSS watcher, background queue)
bin/dev                          # uses Procfile.dev via foreman

# Or run individually:
bin/rails server                 # web server on port 3000
bin/rails dartsass:watch         # CSS compilation (dartsass-rails)
bin/rails solid_queue:start      # background job queue

# Database
bin/rails db:migrate             # primary DB migrations
bin/rails db:migrate:queue       # queue DB migrations (db/queue_migrate/)

# Tests
bin/rails check                  # EVERYTHING: minitest + RSpec API specs + swagger freshness (no system tests)
bin/rails test                   # all minitest tests (does NOT run the RSpec API specs)
bin/rails test test/models/      # test a directory
bin/rails test test/models/user_test.rb        # single file
bin/rails test test/models/user_test.rb:42     # single test by line
bin/rails test:system            # Capybara system tests

# Code quality
bundle exec rubocop              # linting (rubocop-rails-omakase)
bundle exec brakeman             # security analysis

# API specs & Swagger docs (RSpec + rswag)
bundle exec rspec spec/requests/api/v1/          # run API tests
bundle exec rails rswag:specs:swaggerize         # regenerate swagger/v1/swagger.yaml
bin/rails swagger:verify         # fail if swagger.yaml is stale (also part of `check`)
```

## Architecture

### Two Operating Modes

The system operates in either **contest mode** or **group mode** (configured via `GraderConfiguration`). This affects how problems are scoped and presented to users.

### Key Domain Models

- **User** — has roles (admin, group_editor, reporter) via HABTM; scoped access to problems/contests
- **Problem** — programming problems with test cases, statements, attachments
- **Dataset** — test case sets for a problem (one is "live" at a time); contains Testcase records
- **Submission** — user code submissions; tracked through states: submitted → evaluating → done/error. `grader_comment` is a per-testcase result string (one character per testcase, in `Dataset#testcases.display_order`):
  - `P` — pass (full credit)
  - `T` — time limit exceeded
  - `x` — invalid operation (segmentation fault) or memory limit exceeded
  - `-` — wrong answer
  - `s` — partial credit on this testcase
  When diffing two grading runs (e.g. legacy vs migrated), only `T -> P` and `x -> P` transitions are usually benign — they reflect machine-speed or memory differences. All other transitions reflect real score changes worth investigating.
- **Contest** — time-bound competitions with users and problems
- **Group** — organizes problems and users (alternative to contests)
- **Job** — grading jobs (compile, evaluate, score) processed by external judge workers
- **Comment** — supports LLM-assisted hints with cost tracking
- **AuditLog** — polymorphic audit trail for changes to `GraderConfiguration`, `Contest`, `Problem`, `Dataset`, `Testcase`, `ContestProblem`, `ContestUser` (see "Audit Logging" below)

### JSON API (`/api/v1/`)

- Lives in `app/controllers/api/v1/`, routes under `namespace :api / :v1`
- **JWT auth** via `Authorization: Bearer <token>` (session auth is NOT used)
- **Must reuse existing model authorization** (`User#problems_for_action`, `User#can_view_testcase?`, `User#can_view_submission?`, etc.) — never duplicate business logic in API controllers
- **rswag** specs in `spec/requests/api/v1/` double as tests and Swagger docs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cafe-grader-team/cafe-grader-web](https://github.com/cafe-grader-team/cafe-grader-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
