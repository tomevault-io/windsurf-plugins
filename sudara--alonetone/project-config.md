---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **⚠️ This repo is public (github.com/sudara/alonetone).** This file is committed and world-readable. Do not add secrets, private URLs, internal-only hostnames, customer data, unpublished security details, or anything else that shouldn't live on the open internet. Treat it like a README.

## Stack

Rails 8.1 on Ruby 4.0, MySQL, Sidekiq, Active Storage (S3 in prod, local in dev), Hotwire Turbo + Stimulus 3, Propshaft + jsbundling-rails (esbuild) for assets, dartsass-rails for SCSS with a dark/light theme system. Audio playback uses the custom `@alonetone/stitches` library. Auth is Authlogic (with scrypt); spam detection is Rakismet (Akismet).

## Development commands

```bash
rails setup              # copies *.example.yml configs, touches JS stubs, runs db:setup
bin/dev                  # boot server (Rails + esbuild watch + dart-sass via foreman)
bundle exec guard start  # watch-mode RSpec — runs the matching spec when a file changes
bundle exec rspec spec/features/home_page_spec.rb  # run a single spec
bundle exec rspec --exclude-pattern "spec/features/**/*_spec.rb"  # fast, non-browser suite
bundle exec rubocop      # linter (rules in .rubocop.yml — loose: Metrics/* bumped to 500)
bundle exec rails db:reset  # reset DB and reload seeds (programmatic, not a dump)
```

Seed logins: `owner`, `moderator`, `musician`, `marieh`, `carole`, `petere`; password `testing123` for all.

Feature specs run in headless chromium via Playwright (`capybara-playwright-driver`), registered as `Capybara.default_driver = :alonetone` in `spec/rails_helper.rb`. Run `HEADED=1 bundle exec rspec spec/features/...` to watch the browser. The `playwright` npm package and `npx playwright install chromium` are required dev dependencies. Fixtures are global (`config.global_fixtures = :all`) and fixtures + support helpers get auto-included.

CI runs two jobs in parallel: "Normal Specs" (everything except `spec/features`) and "Feature Specs & Percy" (features with visual-regression snapshots via `npx percy exec`). Percy requires `PERCY_TOKEN`.

## Domain vocabulary (has historical baggage)

- **Asset** = an mp3 (audio track). The model is the hot center of the app.
- **Track** = a join between an `Asset` and a `Playlist` (the join row, not the audio).
- **Playlist** = either an album (`is_mix: false`) or a mix (`is_mix: true`). Every user has exactly one `is_favorite: true` playlist that gets appended to when they "heart" something.
- **Home page** routes to `assets#latest`.
- Many views live in `app/views/shared/` — more than ideal.

## Architecture

**Routing shape.** `config/routes.rb` mounts most user-facing URLs *under the user's login slug* — `/:login/tracks/:id`, `/:login/playlists/:id`, `/:login/history`, `/:login/comments`, `/:login/toggle-follow`. This means `find_user` / `find_asset` in `ApplicationController` resolve records by slug (`login`, `permalink`) rather than numeric ID. The `Slugs` concern (`app/models/concerns/slugs.rb`) auto-generates unique slugs scoped per-user. `/admin/*` is a separate namespace for moderation.

**Soft deletion is pervasive.** The `SoftDeletion` concern (`app/models/concerns/soft_deletion.rb`) adds a `default_scope` that hides `deleted_at IS NOT NULL` rows — so queries silently exclude soft-deleted records unless you call `.with_deleted` or `.only_deleted`. `Asset`, `User`, `Playlist` all include it. Records destroyable after 30 days via `PurgeEligibleRecordsJob` / `destroy_deleted_older_than_30_days`. 404 handlers in `ApplicationController` explicitly check `with_deleted` to give friendlier "recently deleted" messages.

**Spam & abuse.** `Rakismet::Model` is mixed into `User`, `Asset`, and `Comment` with different `comment_type` values (`signup`, `mp3-post`, `comment`). The admin controllers expose `spam`/`unspam` member actions. `PreventAbuse` controller concern and a rudimentary banned-words list force-mark submissions as spam. IP-based spam sweeps: `Admin::UsersController#mark_all_users_with_ip_as_spam` + `MarkAllUsersWithIpAsSpam` job.

**Storage abstraction.** `app/models/storage/` wraps Active Storage URL generation. `Storage::Location` picks between direct S3, signed CloudFront (for originals), or Fastly (for image variants) based on `Rails.application.fastly_enabled?` / `cloudfront_enabled?` / `remote_storage?` predicates defined in `config/application.rb`. Treat `Location#url` as a lazy promise — views may evaluate late. Variant processing uses libvips (`config.active_storage.variant_processor = :vips`); analyzers and previewers are disabled.

**Uploads.** `Upload` (service object, `app/models/upload.rb`) accepts an array of `ActionDispatch::Http::UploadedFile`s, dispatches each to `Upload::Mp3File` or `Upload::ZipFile` based on mime type (shells out to `file --mime-type`), and builds `Asset` + `Playlist` records in batch. Import is best-effort: it saves whatever parsed successfully and surfaces the rest as validation errors. Audio validation is enforced on `Asset#audio_file` (<60 MB, `audio/mpeg|mp3|x-mp3`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sudara/alonetone](https://github.com/sudara/alonetone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
