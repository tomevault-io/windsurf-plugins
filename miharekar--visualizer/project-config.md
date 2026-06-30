---
trigger: always_on
description: This file provides guidance to AI coding agents working with this repository.
---

# Visualizer

This file provides guidance to AI coding agents working with this repository.

## What is Visualizer?

Visualizer is a coffee telemetry and sharing app. It ingests brew history files and API uploads from Decent, Beanconqueror, Gaggiuino, GaggiMate, Smart Espresso Profiler (CSV), and other sources, then charts flow/pressure/temperature curves. Users manage roasters and coffee bags, add tasting notes, share shots publicly or with the community, and unlock metadata/tagging/older history with premium. An OAuth + basic-auth API allows third-party clients to upload and fetch shot data.

# Token efficiency

Respond like smart caveman. Cut all filler, keep technical substance.
- Drop articles (a, an, the), filler (just, really, basically, actually).
- Drop pleasantries (sure, certainly, happy to).
- No hedging. Fragments fine. Short synonyms.
- Technical terms stay exact. Code blocks unchanged.
- Pattern: [thing] [action] [reason]. [next step].

## Development Commands

### Setup and server

```bash
bin/setup              # install gems, prepare DB, clear logs/tmp, start bin/dev unless --skip-server
bin/dev                # Overmind: Rails server on PORT (3000 default), tailwind watcher, SolidQueue worker
PORT=4000 bin/dev      # override port
bin/jobs               # run SolidQueue supervisor/worker if you want it outside bin/dev
```

Development URL: http://localhost:3000  
Create a user at `/registrations/new` (Turnstile verification only runs in production).

### Testing

```bash
bin/rails test                          # full Minitest suite (parallelized)
bin/rails test test/models/shot_test.rb # single file
bin/rails test test/models/shot_test.rb:123 # single test by line number
bin/rails test test/models/shot_test.rb -n /test_name/ # single test by name
bin/ci                                  # CI pipeline (style, security, tests, seeds)
env RAILS_ENV=test bin/rails db:seed:replant # must pass in CI
PARALLEL_WORKERS=1 bin/rails test       # use if parallelization causes issues
```

`bin/ci` runs: bin/setup --skip-server, RuboCop, bundler-audit, importmap audit, Brakeman, Gitleaks audit, Rails tests, and `db:seed:replant`.
Parser fixtures for upload/tests live in `test/files/`.

### Database

```bash
bin/rails db:prepare   # create DBs and run migrations
bin/rails db:migrate   # apply new migrations
bin/rails db:reset     # drop/create/migrate primary DB
bin/rails db:seed      # load seed data
```

Primary data lives in Postgres (`primary`).

### Other utilities

```bash
bin/rubocop            # rubocop-rails-omakase baseline
bin/brakeman --quiet --no-pager --exit-on-warn --exit-on-error
bin/bundler-audit
bin/importmap audit
bin/gitleaks-audit
bin/rails tailwindcss:watch # usually handled by bin/dev
```

### Build and lint quick picks

```bash
bin/rubocop            # Ruby lint/style
bin/rails test         # app tests
bin/ci                 # full CI pipeline
```

## Architecture Overview

### Data and storage

- ActiveRecord primary DB is Postgres.
- ActiveStorage uses local disk in development/test; S3 buckets configured in `config/storage.yml` for production.
- Solid Cache backs Rails caching; AppSignal handles monitoring/tracing.
- All tables use UUID primary keys (see `db/schema.rb`).

### Concerns & modules

- Shared model behavior lives in `app/models/concerns`; use `ActiveSupport::Concern` when you need `included` blocks or `class_methods` (`Airtablable`, `Sluggable`, `Squishable`).
- Pure helper mixins can remain as plain modules in concerns (`Bsearch`, `DateParseable`, `ShotPresenter`).
- Class-scoped modules live under the class name in `app/models/<class>/` and are included in the class (e.g., `Shot::Jsonable`, `ShotInformation::Profile`, `ShotChart::Process`).
- Controller mixins live in `app/controllers/concerns` (`Authentication`, `Authorization`, `Filterable`, `Paginatable`); prefer these over fat controllers.
- Integration/service wrappers live in `app/lib` (autoloaded); keep `lib/misc` for experiments only.

### Domain model

- `User` has password + optional WebAuthn passkeys, premium flags, time zone/skin preferences, and many `Session` records.
- `Shot` is the core brew record with profile metadata, attachments, tags, and optional `ShotInformation` JSON for charting.
- `ShotInformation` stores parsed brew data, profile fields, and parser metadata (`parser_name` detects Decent/Beanconqueror/Gaggiuino/GaggiMate/SEP CSV).
- `CoffeeBag` belongs to a `Roaster`; canonical roaster/bag records back autocomplete and community sharing.
- Coffee bag freezer lifecycle uses `frozen_date` and `defrosted_date` naming consistently across model/controller/views/API.
- `SharedShot` issues short codes for public shares and Beanconqueror deep links; `ShotTag`/`Tag` provide tagging.
- `Update`, `Stats`, `YearlyBrew`, and `Community` controllers back the public change log and discovery pages.

### Ingestion & API

- Shot uploads go through `Shot.from_file` (dispatches parsers under `app/models/parsers`) from the web UI or `/api/shots/upload`.
- API controllers (`/api/shots`, `/api/roasters`, `/api/coffee_bags`, `/api/me`) accept Doorkeeper OAuth tokens (`upload`/`write` scopes) or HTTP basic auth; session cookies also work for logged-in users.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miharekar/visualizer](https://github.com/miharekar/visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
