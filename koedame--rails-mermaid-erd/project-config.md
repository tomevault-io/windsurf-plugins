---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Ruby gem that adds a `mermaid_erd` Rake task to a host Rails app. The task introspects the app's ActiveRecord models and emits a single self-contained HTML file at `<app_root>/mermaid_erd/index.html` containing a Vue 3 + Mermaid.js editor (Tailwind + i18n en/ja, all inlined — no build step). The generated file is meant to be opened directly in a browser or hosted statically.

## Commands

All commands run inside the dev container (`docker compose exec devcontainer ...`) unless noted. The dummy app and CI both run on PostgreSQL 14 (`pg` is the only DB driver wired into the dev container), so the test DB must be Postgres.

```bash
# First-time setup (after `docker compose up -d`):
docker compose exec -w /workspace/spec/dummy devcontainer bundle exec rails db:setup RAILS_ENV=test

# Run the full RSpec suite (coverage via SimpleCov drops in /coverage):
docker compose exec devcontainer bundle exec rspec

# Run a single spec file or example:
docker compose exec devcontainer bundle exec rspec spec/rails-mermaid_erd/builder/model_data_spec.rb
docker compose exec devcontainer bundle exec rspec spec/rails-mermaid_erd/builder/model_data_spec.rb:6

# Lint (StandardRb — enforced in CI):
docker compose exec devcontainer bundle exec standardrb
docker compose exec devcontainer bundle exec standardrb --fix

# Exercise the generator end-to-end against the dummy app — writes spec/dummy/mermaid_erd/index.html:
docker compose exec -w /workspace/spec/dummy devcontainer bundle exec rails mermaid_erd RAILS_ENV=test

# Run rspec against one row of the matrix (Rails 7.2 example):
docker compose exec -e BUNDLE_GEMFILE=/workspace/gemfiles/rails_7_2.gemfile devcontainer bundle exec rspec

# Regenerate gemfiles/*.gemfile after editing Appraisals at the repo root:
docker compose exec devcontainer bundle exec appraisal install
```

CI runs natively on GitHub Actions via `ruby/setup-ruby` + `services.postgres` (no `compose.ci.yml` — that file is gone). `compose.yml`/`Dockerfile` are local-dev only and pinned to a single Ruby; the dev container can only exercise matrix rows whose Rails supports that Ruby. The full Ruby × Rails matrix lives in `Appraisals` (repo root) and `.github/workflows/run-test.yml` — keep both in sync. The `gemfiles/*.gemfile` files are committed; their lockfiles are gitignored and resolved fresh per Ruby in CI. Regenerate the gemfiles with `bundle exec appraisal install` after editing `Appraisals`.

## Architecture

- **`lib/rails-mermaid_erd.rb`** — declares the `mermaid_erd` Rake task. It calls `Builder.model_data`, then evaluates `lib/templates/index.html.erb` with `version`, `app_name`, `logo`, and `result` (the schema dump) in scope, and writes the rendered HTML to the path from `Configuration#result_path`. ERB binding is the entire integration contract between Ruby and the front-end.

- **`lib/rails-mermaid_erd/builder.rb`** — the only nontrivial Ruby. Calls `Rails.application.eager_load!`, walks `ActiveRecord::Base.descendants`, and emits `{Models: [...], Relations: [...]}`. Two things to know before touching it:
  1. **Relation deduplication is direction-aware, but the `Line`-style filter is selective.** For each model it iterates `has_many`, `has_and_belongs_to_many`, `belongs_to`, `has_one` and looks up a reverse pair in `Relations` by matching `LeftModelName`/`RightModelName`. The `has_many` and `has_one` branches *also* filter on `Line` style (`".."` for `through:`, `"--"` otherwise — `builder.rb:48-54, 120-126`), so direct and `through:` associations don't collapse into each other. The `belongs_to` (`builder.rb:94`) and `has_and_belongs_to_many` (`builder.rb:76`) branches match on names only — don't add a `Line` filter there, it would break HABTM and BT-vs-pre-existing-HM merges. When a reverse pair is found, the entry is *merged* (the new association name is appended to `Comment`, and cardinality glyphs may upgrade `||` → `|o` for optional `belongs_to`).
  2. **Model name resolution** goes through `get_reflection_model_name`, which honors `class_name:`, then `through:` + `source:`, else falls back to `reflection.class_name`. The dummy app deliberately exercises all three (`Author` is `users`, `comment_posts` is `has_many :through`, `images` uses `class_name: "UserImage"`).

  HABTM join tables emitted by Rails with `HABTM_` in the name and tables that don't exist yet are skipped — don't add filtering elsewhere.

- **`lib/rails-mermaid_erd/configuration.rb`** — reads `config/mermaid_erd.yml` from the host app via `Rails.root`. Only `result_path` exists today; defaults to `mermaid_erd/index.html`. Keep new keys backwards-compatible (the merge happens on top of a hardcoded default hash).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koedame/rails-mermaid_erd](https://github.com/koedame/rails-mermaid_erd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
