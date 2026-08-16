---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Semantic Logger is a Ruby gem: a feature-rich, multi-destination logging framework and replacement for Ruby & Rails loggers. Its defining trait is **asynchronous logging** — log events are pushed onto an in-memory queue serviced by a background thread, so the application is not blocked while logs are written to one or more destinations ("appenders").

Rails users should use the sister gem `rails_semantic_logger`, not this gem directly.

**Sister gem version lockstep:** `rails_semantic_logger` is locked to the same major version of this gem. As of v5 it has been upgraded in lockstep: it is now on v5.0.0 and its gemspec pins `semantic_logger ">= 5.0"`. Major-version changes here must continue to be mirrored there. The v5 internal refactors (the `QueueProcessor` extraction and the removal of `Appender::AsyncBatch`) have been accounted for; the sister gem does not reference `AsyncBatch` directly. The sister gem lives at `/Users/reidmo/src/rails_semantic_logger` (when checked out locally).

## Public interface

The **only** public-facing interface is the `SemanticLogger` module itself ([lib/semantic_logger/semantic_logger.rb](lib/semantic_logger/semantic_logger.rb)): its module methods (`SemanticLogger[...]`, `add_appender`, `default_level=`, `tagged`, `silence`, `flush`, `reopen`, etc.) plus the `Loggable` mixin. Everything else (`Logger`, `Base`, `Processor`, `Subscriber`, `Appender::*`, `Formatters::*`, `Log`, ...) is **internal** and may be refactored freely, as long as the public interface keeps working.

The one wrinkle: once a `Logger` instance has been handed back (via `SemanticLogger['ClassName']` or the `logger` method from `Loggable`), that instance's API (`info`, `measure_info`, `tagged`, `level=`, ...) is also part of the public contract. But end users never need to know *how* a `Logger` is constructed — it is always obtained through the `SemanticLogger` module or the `Loggable` mixin, so the constructor and the class itself remain internal.

When changing internal classes, the bar is: **do not break any existing public-facing interface** (the `SemanticLogger` module methods or a returned `Logger`'s methods).

The Ruby `Logger` compatibility aliases in [lib/semantic_logger/concerns/compatibility.rb](lib/semantic_logger/concerns/compatibility.rb) (`<<`, `add`, `progname`, `sev_threshold`, `silence_logger`, ...) are **not** part of the published public interface. They exist solely so Semantic Logger can stand in for the Ruby/Rails loggers (Rails and libraries like Active Record's session store call them), so they must keep working for that purpose, but do not document, extend, or promote them as public API.

## Commands

```bash
bundle install              # install dependencies
bundle exec rake            # run the full test suite (the default task)
bundle exec rake test       # same as above
bundle exec rubocop         # lint
LOGGER_SYNC=1 bundle exec rake   # run tests in synchronous mode (no background thread)
```

Run a single test file or test:

```bash
bundle exec ruby -Itest test/logger_test.rb
bundle exec ruby -Itest test/logger_test.rb -n /pattern/
```

Some appender tests need MongoDB. CI runs against Ruby 3.2–4.0 with a MongoDB service on `127.0.0.1:27017` (`MONGO_HOST` env var). Use `docker compose up` (see `docker-compose.yaml`) to start a local MongoDB container; the tests themselves run on the host.

The minimum supported Ruby is 3.2 (as of v5; see `gemspec` and `.rubocop.yml`'s `TargetRubyVersion`) — do not use syntax newer than that in `lib/`.

## Architecture

The logging pipeline has four layers. Understanding the hand-off between them is the key to this codebase:

1. **`SemanticLogger::Logger`** ([lib/semantic_logger/logger.rb](lib/semantic_logger/logger.rb)) — what application code holds (one per class, via `SemanticLogger['ClassName']` or the `Loggable` mixin). `logger.info(...)` etc. build a `Log` object and hand it to the global processor. There is **one shared processor** for the whole process (`Logger.processor`), not one per logger.

2. **`SemanticLogger::Base`** ([lib/semantic_logger/base.rb](lib/semantic_logger/base.rb)) — abstract superclass of both `Logger` and `Subscriber`. It metaprograms the per-level methods (`debug`/`info`/`warn`/..., plus `measure_*` and `benchmark_*`) from `Levels::LEVELS`, and contains the argument-parsing logic in `log_internal` / `measure_internal` that turns the flexible call signatures (message, payload hash, exception, block) into a populated `Log`.

3. **`SemanticLogger::Processor`** ([lib/semantic_logger/processor.rb](lib/semantic_logger/processor.rb)) — a singleton that **is** an `Appender::Async`. It owns the background thread and the queue. It fans each `Log` out to the `Appenders` collection. `SyncProcessor` ([lib/semantic_logger/sync_processor.rb](lib/semantic_logger/sync_processor.rb)) is the drop-in replacement used when `SemanticLogger.sync!` is called (or `require "semantic_logger/sync"`), which logs inline on the calling thread — used heavily in tests.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reidmorrison/semantic_logger](https://github.com/reidmorrison/semantic_logger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
