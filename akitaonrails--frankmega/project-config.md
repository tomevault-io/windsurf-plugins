---
trigger: always_on
description: validates :field, presence: true, ...
---

# FrankMega - Development Guide

## Project Overview

Security-hardened, self-hosted file sharing service. Users upload files, get time-limited shareable links with download counters. Deployed as a Docker container behind Cloudflare Tunnel at `frankmega.example.com`.

**Stack:** Ruby 3.4.8, Rails 8.1.2, SQLite3, Tailwind CSS v4, Propshaft, Importmap, Solid Queue/Cache/Cable, Puma + Thruster.

## Commands

```bash
# IMPORTANT: All bundle commands require this env var (read-only ~/.bundle workaround)
export BUNDLE_USER_HOME=/tmp/bundler_home

# Run tests
bundle exec rails test

# Run linters
bundle exec rubocop
bundle exec brakeman --no-pager --quiet
bundle exec bundler-audit check

# Start dev server
bin/dev

# Compile Tailwind manually
bundle exec rails tailwindcss:build

# Database
bundle exec rails db:migrate
bundle exec rails db:seed         # Seeds AllowedMimeType defaults
RAILS_ENV=test bundle exec rails db:migrate

# Docker
docker compose build
docker compose up
```

## Architecture

### Database

SQLite3 with separate databases in production for app, cache, queue, and cable (see `config/database.yml`). All databases live in `storage/`.

### Authentication

Built on Rails 8 built-in auth generator (`app/controllers/concerns/authentication.rb`). Every controller inherits `Authentication` through `ApplicationController`. Key methods:

- `current_user` - returns the logged-in User or nil
- `authenticated?` - helper method available in views
- `allow_unauthenticated_access` - class method to skip auth on specific actions
- `start_new_session_for(user)` - creates session record and sets signed cookie
- `require_admin` - redirects non-admin users

Auth flow: password login -> optional 2FA challenge -> session cookie. Banned users are rejected at `resume_session` level.

### Authorization

Two roles: `"admin"` and `"user"`. Check with `current_user.admin?`. Admin controllers inherit from `Admin::ApplicationController` which enforces `require_admin` as a `before_action`.

### File Storage

Active Storage with local disk backend. Production storage path configurable via `STORAGE_PATH` env var (defaults to `storage/uploads`). Max upload size: **1 GB**. Allowed MIME types managed via `AllowedMimeType` model (admin-configurable).

### Background Jobs

Solid Queue (no Redis). Recurring jobs configured in `config/recurring.yml`:
- `CleanupExpiredFilesJob` - every 15 minutes
- `CleanupExpiredBansJob` - every hour

All jobs use `queue_as :default`.

### Real-time

Turbo Streams via Action Cable (Solid Cable). Download notifications broadcast to user-specific channels: `"user_#{user_id}_notifications"`.

## Coding Conventions

### Models

Follow this ordering within model files:

```ruby
class ModelName < ApplicationRecord
  # 1. Associations
  belongs_to / has_many / has_one_attached

  # 2. Encryption
  encrypts :field_name

  # 3. Normalizations
  normalizes :field, with: -> ...

  # 4. Validations
  validates :field, presence: true, ...
  validate :custom_validation, on: :create

  # 5. Callbacks
  before_validation :method, on: :create

  # 6. Scopes
  scope :active, -> { where(...) }

  # 7. Public instance methods
  def active?
  end

  # 8. Private methods
  private
  def generate_something
  end
end
```

Key patterns:
- Use `SecureRandom.urlsafe_base64` for tokens/hashes (16 bytes for invitation codes, 24 bytes for download hashes)
- Use `Time.current` (never `Time.now`) for all time comparisons
- Define both scope and instance method for status checks (e.g., `scope :active` and `def active?`)
- Sensitive fields use `encrypts` (ActiveRecord Encryption) - currently `otp_secret` on User
- State-changing methods end with `!` when using `update!` (e.g., `ban!`, `redeem!`, `enable_otp!`)
- Boolean columns always have `default: false, null: false` in migrations
- Required string/integer columns use `null: false` in migrations
- Add database indexes for columns used in lookups (unique constraints, foreign keys, `expires_at`)

### Controllers

```ruby
class SomeController < ApplicationController
  # 1. Auth configuration
  allow_unauthenticated_access only: %i[show create]

  # 2. Before actions
  before_action :find_resource

  # 3. Actions (new, create, show, edit, update, destroy order)
  def show
  end

  # 4. Private methods
  private
  def find_resource
  end

  def resource_params
    params.require(:model).permit(:field1, :field2)
  end
end
```

Key patterns:
- Scope queries to `current_user` for data isolation: `current_user.shared_files.find(params[:id])`
- Use `if/elsif/else` blocks instead of early `return` + `render` (rubocop enforces no redundant returns)
- Public endpoints use `allow_unauthenticated_access`
- Admin controllers inherit `Admin::ApplicationController` (not `ApplicationController` directly)
- Admin namespace: `module Admin; class FooController < Admin::ApplicationController`
- Strong parameters: always use `params.require(:model).permit(...)` - never `params.permit` at top level
- Use `status: :unprocessable_entity` when re-rendering forms on validation failure
- Use `status: :see_other` on destroy redirects
- Background work goes in jobs, not inline in controllers

### Routes

- Public download URLs use short paths: `/d/:hash`
- Admin routes namespaced under `/admin`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akitaonrails/FrankMega](https://github.com/akitaonrails/FrankMega) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
