---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`rocketjob` is a Ruby gem: a distributed, MongoDB-backed batch processing system. Jobs are Mongoid documents persisted to MongoDB; servers pull queued jobs and run them across worker threads. There is no Rails app here, only the library plus its test suite. The web UI (Mission Control) and Tabular plugins live in separate gems.

The primary public interface is `RocketJob::Job`; capabilities are added by mixing in modules such as `RocketJob::Batch`. Treat that surface (subclassing `Job`, opt-in mixins, typed fields, `#perform`, queuing/downloading, documented state transitions) as the stable contract; everything under `Plugins`, `Sliced`, and the runtime classes is internal.

## Why this design

Context worth knowing before changing core behavior:

- **Two tiers of jobs.** Simple jobs (`RocketJob::Job`) are conventional background jobs like other frameworks offer. Batch jobs (`include RocketJob::Batch`) are the real point of Rocket Job: a single job's input is uploaded into a dynamically created MongoDB collection, split into slices, and processed concurrently across thousands of workers (often Docker containers). Output is written back to MongoDB the same way.
- **Why MongoDB.** Its atomic `find_and_modify` lets thousands of nodes claim work without colliding, and it spills from memory to disk, which is essential for very large files. Rocket Job was built because Sidekiq/Redis could not scale this way (Redis was single-threaded and could not overflow to disk). **AWS DocumentDB is NOT compatible**: the `Event`/`Subscriber` pub-sub mechanism (`lib/rocket_job/event.rb`) requires a *tailable capped collection*, and DocumentDB still does not support capped collections (verified June 2026 against AWS docs). `docs/installation.md` is correct to say so.
- **Why Mongoid.** Jobs are documents with real, typed, validated fields, not an untyped hash of arguments.
- **Backward compatibility is a priority.** Only break it in a major release, ideally with a deprecation path first. Most forced changes come from breaking changes in MongoDB/Mongoid. When adding persisted fields to a plugin, give them defaults so existing jobs in the database still load.

A class diagram of the core relationships, and the architecture overview, is in [docs/architecture.md](docs/architecture.md) (published as `architecture.html`). `CONTRIBUTING.md` only points to it.

## Commands

Tests require a running MongoDB on `127.0.0.1:27017` (see `docker-compose.yml`: `docker compose up -d`). Test config is `test/config/mongoid.yml` and the suite drops/recreates collections on load.

- Run full suite (against current `Gemfile`): `bundle exec rake test`
- Run one test file: `bundle exec ruby -Itest -Ilib test/job_test.rb`
- Run one test by name: `bundle exec ruby -Itest -Ilib test/job_test.rb -n /pattern/`
- Lint: `bundle exec rubocop` (config in `.rubocop.yml`)
- Test logs go to `test.log` (not stdout); use it to debug failures.

`rake` with no args runs the full matrix via Appraisal, not a single test run. Don't use bare `rake` for quick iteration.

### Multi-version testing (Appraisal)

CI runs against multiple Mongoid/Rails/Ruby combinations defined in `Appraisals` (Mongoid 8.1, 9.0, 9.1) with generated gemfiles in `gemfiles/`. To target one combination:

- `BUNDLE_GEMFILE=gemfiles/mongoid_9.1.gemfile bundle exec rake test`
- Regenerate gemfiles after editing `Appraisals`: `bundle exec appraisal install`

### Test conventions and gotchas

The suite is Minitest with `minitest/spec` `describe`/`it` blocks nested inside `class FooTest < Minitest::Test`. Heavy use of `Minitest::Mock`, `.stub`, and `minitest-stub_any_instance`. Useful things learned writing tests:

- **Coverage.** SimpleCov runs automatically (started in `test_helper.rb`). After a run, overall % prints to stdout and `coverage/.resultset.json` holds per-file line hits — parse it to find the least-covered files and exact missing line numbers. `coverage/index.html` is the browsable report.
- **Shared class-level state must be reset between tests.** `Supervisor` keeps `@shutdown`/`@event` (`Concurrent::Event`) as class instance vars; `Event` keeps a class-level `@subscribers` map; `Subscriber` has `@test_mode`. Reset these in `before`/`after` or tests leak into each other. See `test/supervisor_test.rb` / `test/event_test.rb` for the reset helpers.
- **Autoload ordering.** Constants are `autoload`ed in `lib/rocketjob.rb`. Some files trigger circular loads when referenced first in isolation — e.g. referencing `RocketJob::WorkerPool` before `RocketJob::Supervisor` fails because `worker_pool.rb` requires `supervisor/shutdown`. Fix by `require`ing the dependency (e.g. `require "rocket_job/supervisor"`) at the top of the test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reidmorrison/rocketjob](https://github.com/reidmorrison/rocketjob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
