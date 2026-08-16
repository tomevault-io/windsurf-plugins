---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Rails Semantic Logger is a Rails engine gem that **monkey-patches Rails to replace its loggers and log subscribers with [Semantic Logger](https://logger.reidmorrison.com/)**, so that Rails produces structured logs directly instead of human-readable text that downstream systems have to parse. Rather than parsing Rails' text output (Started, Processing, Completed, Rendered, SQL, etc.), it replaces the built-in `ActiveSupport::LogSubscriber`s (and other loggers) wherever possible, emitting a message plus a structured payload that can be rendered as text, color, or JSON for centralized logging.

This gem only adapts Rails to Semantic Logger; the actual logging engine, appenders, and formatters live in the separate `semantic_logger` gem (a runtime dependency).

### The core maintenance challenge: tracking Rails' log subscribers

Rails' own log subscribers change between Rails versions, sometimes even in minor/patch releases. Because this gem reimplements those subscribers to emit structured payloads, that coupling makes it **inherently brittle**. The central, recurring maintenance task is:

1. Look at the log subscriber source for each supported Rails version (ActionController, ActionView, ActiveRecord, ActiveJob, ActionMailer, etc.).
2. Bring any upstream changes across into this gem's corresponding subscriber under `lib/rails_semantic_logger/<component>/log_subscriber.rb`.
3. Replace the upstream **text** output with **hash / structured** logging (message + payload) while preserving the new behavior.

When touching a subscriber, diff it against the matching Rails version's subscriber to confirm parity. To make this easier, every Rails component subscriber (action_controller, action_view, active_record, active_job, action_mailer) carries a comment block at the top listing the upstream Rails source URLs for each supported version (one per `*-stable` branch), so a maintainer can jump straight to the canonical source to compare. Keep those links current when the supported Rails versions change; `lib/rails_semantic_logger/active_record/log_subscriber.rb` shows the pattern. (`solid_queue/log_subscriber.rb` has no such block because SolidQueue is not a Rails component.)

As of Rails 8.1 there is a **second** source to cross-reference: each component now also ships a `*/structured_event_subscriber.rb` (e.g. `active_record/structured_event_subscriber.rb`), subclassing `ActiveSupport::StructuredEventSubscriber`. These are Rails' own structured-event emitters (message + hash, via `Rails.event` / the `ActiveSupport::EventReporter`), so they are the authoritative, Rails-maintained source for **field names and payload shape** when adding structured fields here. They are a *reference only*, not something this gem currently uses: Rails feeds two parallel pipelines from the same `ActiveSupport::Notifications` events, the classic `LogSubscriber` (text -> `Rails.logger`, which this gem swaps) and the new `StructuredEventSubscriber` (structured -> `Rails.event`). `Rails.event` ships with **no subscribers** by default, so the structured subscribers are dormant and do not conflict with our swapped subscribers. Note also that some structured events are emitted via `emit_debug_event` (only when `Rails.event.debug_mode?`, default development-only), so they are not a production-complete substitute. When syncing a subscriber, diff against **both** the classic `log_subscriber.rb` (for parity/behavior across 7.2/8.0/8.1) and, on 8.1, the `structured_event_subscriber.rb` (for the canonical field names).

This gem does **not** switch to `StructuredEventSubscriber`: it only exists in 8.1 (we still support 7.2/8.0), covers only a subset of components (and none of the non-Rails integrations such as Sidekiq/Mongoid/SolidQueue), and several events are debug-mode-only. Routing `Rails.event` into Semantic Logger would be an additive, opt-in bridge for a future major, not a replacement for swapping the `LogSubscriber`s.

### Relationship to semantic_logger

- The companion `semantic_logger` gem is usually checked out locally at **`../semantic_logger`** — reference it when you need to understand appender/formatter behavior.
- The two libraries are kept **in lock step at the major-version level**, so breaking changes in `semantic_logger` are paired with a matching `rails_semantic_logger` major version. Keep this in mind when bumping versions or relying on new `semantic_logger` features.

## Commands

Tests run via Appraisal across multiple Rails versions. Gems install into the global gem list.

```bash
appraisal install                  # install gems for every appraisal (regenerates gemfiles/)
rake                               # default task: run tests against ALL appraisals
rake test                         # run tests once against the current Gemfile
appraisal rails_8.0 rake          # tests for one Rails version
appraisal rails_8.0 ruby test/controllers/articles_controller_test.rb            # single test file
appraisal rails_8.0 ruby test/controllers/articles_controller_test.rb -n "/shows new article/"   # single test by name
rubocop                           # lint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reidmorrison/rails_semantic_logger](https://github.com/reidmorrison/rails_semantic_logger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
