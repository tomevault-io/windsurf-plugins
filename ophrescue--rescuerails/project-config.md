---
trigger: always_on
description: RescueRails is the web app for Operation Paws for Homes, a dog/cat rescue.
---

# RescueRails (Operation Paws for Homes)

## App overview

RescueRails is the web app for Operation Paws for Homes, a dog/cat rescue.
It has a public site (adoption listings, applications, donations) and an
internal admin/staff system (`app/controllers/dogs/manager`,
`app/controllers/cats/manager`, dashboards) for managing dogs, cats,
adopters, volunteers, and staff. No admin-engine gem (ActiveAdmin/RailsAdmin)
is used — the admin UI is hand-built.

The app is ~13 years old (`db/migrate` goes back to 2011) and has never, in
its history, gone past Rails 6.1 — see "Upgrade roadmap" below.

**Stack:**
- Ruby 3.0.5, Rails 7.1.6 (as of the pass recorded below)
- PostgreSQL, `pg` gem
- Auth: **Clearance** (not Devise) — see `config/initializers/clearance.rb`
- File uploads: **kt-paperclip** (not ActiveStorage, though activestorage
  ships with Rails regardless)
- Background jobs: **DelayedJob** (not Sidekiq)
- Asset pipeline: **dual** Sprockets + Webpacker 5 (deprecated/unmaintained
  but still functional)
- Production app server: **Unicorn** (prod-only Gemfile group); **Puma**
  only in dev/test — a pre-existing inconsistency, not yet reconciled
- Deploy: Capistrano (not containers/Kamal) — the `.devcontainer/` setup
  is local-dev-only, unrelated to how production is deployed
- Tests: RSpec (~108 spec files across models/controllers/requests/
  features/mailers), FactoryBot, Capybara + Cuprite (headless Chrome)
  for feature/system specs, SimpleCov

**Key directories:** `app/models` (49), `app/controllers` (35, with
`cats/`/`dogs/` namespaces), `app/jobs`, `app/services`, `spec/` (mirrors
app/ plus `spec/features` for Capybara specs), `lib/` (custom autoloaded
code — Clearance guards, password strategy, Paperclip interpolations —
added to autoload/eager_load paths explicitly via
`config.paths.add 'lib', eager_load: true` in `config/application.rb`).

## Working conventions

This app is mid-way through a multi-pass dependency modernization
(everything was badly out of date: EOL Ruby, EOL Rails, deprecated gems).
Ground rules for every pass:

- **Limit changes to what's required for that specific upgrade.** Don't
  opportunistically bump unrelated gems, don't fix unrelated papercuts,
  don't fold in adjacent modernization work unless the current pass is
  literally blocked without it. If something forces an out-of-scope
  change, do the minimal version of it and say so explicitly in the
  commit message — don't let it expand further.
- **Rails major-version bumps happen one version at a time** (6.1 → 7.0 →
  7.1/7.2 → 8.0, not 6.1 → 8.0 directly).
- **Don't bump `config.load_defaults` in the same pass as a Rails version
  bump.** This app's `config/application.rb` has been stuck at
  `load_defaults 5.0` since long before this upgrade effort started, so
  catching it up is its own dedicated future pass with its own test
  cycle — not something to absorb silently while also swapping the
  framework version.
- **Verification gate: full RSpec suite, compared against a baseline
  captured before the change.** With ~108 spec files this is cheap enough
  to always run in full rather than by subset. Use `bin/rails` / `bin/rspec`
  binstubs, **not** `bundle exec rails` / `bundle exec rspec` — the latter
  fail with "command not found" in this devcontainer environment for
  unclear reasons; the binstubs work correctly.
- **Scoped `bundle update <gem> <gem> --conservative`, never a bare
  `bundle update`.** Review the resulting `Gemfile.lock` diff line by
  line — if gems outside the intended set move, stop and investigate
  before proceeding; don't wave it through.
- Runtime gem incompatibilities can exist even when `bundle install`
  resolves cleanly — a gem's version constraints not capping a newer
  Rails doesn't mean its code is actually compatible with it. Don't treat
  a successful `bundle update` as proof the app still works; the RSpec
  suite is the real gate.
- **One PR per pass, staged into a few reviewable commits** (e.g. forced
  companion-gem bumps in one commit, the core framework bump in another),
  not one PR per gem and not a single squashed commit.
- When investigating a boot/spec failure, check whether it reproduces on
  the *pre-upgrade* code (`git stash` / a clean checkout of the prior
  commit) before assuming it's caused by the change in progress — several
  issues found during Pass 1 (an empty dead model file, a Node/webpacker
  incompatibility) turned out to be pre-existing and unrelated.

## Upgrade roadmap / log

**Pass 1 — Rails 6.1.7.10 → 7.0.10 (Ruby stays 3.0.5): DONE**
(commits `48293d36`, `e0f56f7b` on `upgrade/rails-7.0`)

- Bumped `audited` (~> 4.5 → ~> 5.0): 4.10.0 hard-capped `activerecord < 6.2`,
  couldn't resolve against Rails 7 at all. Ran `audited:upgrade` generator
  (one migration, reorders two index column pairs on `audits`).
  - Confirmed hard-blocker; not opportunistic.
- Bumped `kt-paperclip` (~> 6 → ~> 7.2, resolved 7.3.0): 6.4.2's
  `AttachmentSizeValidator` referenced `ActiveModel::Validations::
  NumericalityValidator::CHECKS`, which Rails 7.0 renamed to
  `COMPARE_CHECKS`. This doesn't fail at `bundle install` time (no
  version cap in the gemspec), only at runtime as a `NameError` on any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ophrescue/RescueRails](https://github.com/ophrescue/RescueRails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
