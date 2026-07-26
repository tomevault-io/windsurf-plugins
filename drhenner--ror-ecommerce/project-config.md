---
trigger: always_on
description: Test setup, database seeding, and RSpec conventions for the ror_ecommerce Rails app
---


# Test Setup for ror_ecommerce

## Prerequisites

### 1. MySQL must be running

The app uses MySQL via the `mysql2` gem. Start MySQL before running any specs:

```bash
brew services start mysql   # or: mysql.server start
```

If you see `Can't connect to local MySQL server through socket '/tmp/mysql.sock'`, MySQL is not running.

### 2. Cursor sandbox blocks MySQL — always use `required_permissions: ["all"]`

The Cursor sandbox blocks access to the MySQL Unix socket at `/tmp/mysql.sock`. When running specs via the Shell tool, **always** pass `required_permissions: ["all"]` to disable the sandbox. Without this, every spec fails with `ActiveRecord::ConnectionNotEstablished`.

### 3. Database setup (MUST do before running any specs)

The test suite depends on **seed data** that is NOT created by factories. Before running specs for the first time (or after a DB reset):

```bash
RAILS_ENV=test rake db:create db:migrate db:seed
bundle exec rails dartsass:build
```

### Seeded tables (never truncated between tests)

These are populated by `db/seeds.rb` and the test suite assumes they exist:

| Table | Seeded From |
|---|---|
| `countries` | `db/seed/countries.yml` |
| `states` | `db/seed/states.yml` |
| `roles` | `Role::ROLES` |
| `address_types` | `AddressType::NAMES` |
| `phone_types` | `PhoneType::NAMES` |
| `item_types` | `ItemType::NAMES` |
| `deal_types` | `DealType::TYPES` |
| `accounts` | `Account::TYPES` |
| `shipping_rate_types` | `ShippingRateType::TYPES` |
| `shipping_zones` | `ShippingZone::LOCATIONS` |
| `transaction_accounts` | `TransactionAccount::ACCOUNT_TYPES` |
| `return_reasons` | `ReturnReason::REASONS` |
| `return_conditions` | `ReturnCondition::CONDITIONS` |
| `newsletters` | `Newsletter::AUTOSUBSCRIBED` / `MANUALLY_SUBSCRIBE` |
| `referral_bonuses` | `ReferralBonus::BONUSES` |
| `referral_programs` | `ReferralProgram::PROGRAMS` |
| `referral_types` | `ReferralType::NAMES` |

If specs fail with "must exist" errors on `State`, `Country`, `Role`, `ItemType`, `ReferralProgram`, etc. — seed data is missing. Re-run `RAILS_ENV=test rake db:seed`.

### 4. Elasticsearch required for product search specs

Product search uses Searchkick (backed by Elasticsearch). For specs that exercise search, Elasticsearch must be running:

```bash
brew services start elasticsearch
```

Searchkick callbacks are disabled globally in `spec_helper.rb` (`Searchkick.disable_callbacks` in `before(:each)`, re-enabled in `after(:each)`). For specs that need to test actual search results, enable callbacks and reindex within the test:

```ruby
Searchkick.enable_callbacks
Product.reindex
# ... perform search assertions ...
```

## Test Architecture

- **Framework**: RSpec + Mocha (not rspec-mocks). Use `stubs` / `expects`, not `allow` / `expect(...).to receive`.
- **Factories**: FactoryBot. All attribute values MUST use block syntax: `name { "value" }`, never `name "value"`.
- **Database cleaning**: DatabaseCleaner runs between each test. The `before(:suite)` hook calls `trunctate_unseeded` which truncates only non-seed tables.
- **Auth in controller specs**: Use `login_as(user)` or `set_current_user` from `Hadean::TestHelpers`, which stubs `current_user` and creates an Authlogic `UserSession`.
- **Transactional fixtures are OFF** (`use_transactional_fixtures = false`); DatabaseCleaner handles cleanup.

## Common Gotchas

### belongs_to associations
`belongs_to_required_by_default` is set to `false` in `config/application.rb` to match the legacy codebase (app was written for Rails 4/5). If you see "must exist" validation errors, the seed data is likely missing — don't add `optional: true` without checking seeds first.

### ActiveRecord configurations API (Rails 7)
Use `ActiveRecord::Base.connection_db_config.configuration_hash` instead of `ActiveRecord::Base.configurations[Rails.env]`.

### Authlogic 6.x
- `acts_as_authentic` no longer supports `validate_login_field`, `validate_email_field`, or `validates_length_of_password_field_options` setters. Remove them; add explicit validations if needed.
- `password_confirmation` accessor is not auto-provided. The `User` model defines `attr_accessor :password_confirmation` explicitly with `require_password_confirmation = false`.

### Paperclip removed — use Active Storage
The `Image` model uses `has_one_attached :photo`. Call `image.photo_url(:medium)` instead of `image.photo.url(:medium)`. Variants are defined in `Image::IMAGE_STYLES`.

### Payment/CIM stubs
Every test stubs CIM profile methods on `User` and `PaymentProfile` via Mocha in the global `before(:each)`. Don't remove these or payment-related specs will hit external APIs.

### ReferralProgram.current_program
Several specs rely on `ReferralProgram.current_program` returning a record. If this is nil, seed data is missing.

### Asset pipeline (Rails 8.1 / Propshaft)
The app uses Propshaft (not Sprockets) for serving assets, dartsass-rails for SCSS compilation, and importmap-rails. CSS **must be built before running specs** — without it, all admin controller specs fail with `Propshaft::MissingAssetError` for `admin_new.css`. Run `bundle exec rails dartsass:build` after cloning, after SCSS changes, or after a `db:reset`. Compiled CSS is output to `app/assets/builds/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drhenner/ror_ecommerce](https://github.com/drhenner/ror_ecommerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
