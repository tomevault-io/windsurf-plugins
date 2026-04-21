---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Dobase** is a Ruby on Rails 8.1 app that provides user-installable "tools" inside a single workspace. Users add tool instances (boards, chat, mail, files, docs, calendar, room, todos), each optionally shared with collaborators.

### Branding / Configuration

All branding is configurable via environment variables (defaults to "Dobase"):

| Variable | Default | Purpose |
|----------|---------|---------|
| `APP_NAME` | `Dobase` | App name shown in UI, emails, page titles |
| `APP_LOGO_PATH` | `/icon.svg` | Path to logo image (sidebar, auth pages) |
| `APP_HOST` | `localhost:3000` | Host for mailer URLs (production) |
| `APP_FROM_EMAIL` | `notifications@dobase.co` | Sender address for emails |

Config lives in `config/initializers/app_config.rb`. View helpers: `app_name`, `app_logo_path`.

## Development Commands

```bash
bin/dev                    # Start dev server (Rails + Tailwind watcher via foreman)
bin/setup                  # Install deps + prepare DB (--reset to drop/recreate)
bin/rails test             # Run unit/integration tests (Minitest)
bin/rails test:system      # Run system tests (Capybara + Selenium)
bin/rails test test/models/tool_test.rb           # Run single test file
bin/rails test test/models/tool_test.rb:42        # Run single test at line
bin/rubocop                # Lint Ruby (rubocop-rails-omakase)
bin/brakeman --quiet       # Security static analysis
bin/ci                     # Full CI pipeline (setup, lint, audit, tests, seeds)
```

## Deployment

```bash
kamal deploy -d dobase     # Deploy to production (requires -d dobase destination flag)
```

The `config/deploy.yml` contains open-source placeholder values. Real production config lives in `config/deploy.dobase.yml` (the Kamal destination file). Always use `-d dobase` when deploying.

**Tailwind CSS** must be rebuilt after stylesheet changes:
```bash
bundle exec tailwindcss -i ./app/assets/tailwind/application.css -o ./app/assets/builds/application.css
```
The `bin/dev` watcher handles this automatically in development.

## Architecture

### Tool System

Every feature is a **Tool** instance linked to a **ToolType** (slug: `mail`, `boards`, `files`, `chat`, `docs`, `calendar`, `room`, `todos`). `ToolsController#show` redirects to the tool-specific controller based on slug.

**Permissions are binary.** Checked via `ToolAuthorization` concern (`can_access?` / `can_manage?`). Access is tracked in the `collaborators` table with a `role` column:
- **Owner** (`role: "owner"`) — full control, can invite/remove collaborators, promote to owner, rename, delete. Tools can have multiple owners.
- **Collaborator** (`role: "collaborator"`) — full functional access within the tool, cannot delete or re-share

The tool creator is automatically added as an owner collaborator (`after_create :add_creator_as_owner`). Check ownership via `tool.owned_by?(user)` and access via `tool.accessible_by?(user)` — both query the collaborators table. Permissions are explicit and instance-based — never inferred.

### Namespacing Pattern

Models and controllers are namespaced per tool. Models set `self.table_name` explicitly:

```ruby
module Boards
  class Card < ApplicationRecord
    self.table_name = "cards"
  end
end
```

Controllers nest under `Tools::`:
```
app/controllers/tools/boards/cards_controller.rb  → Tools::Boards::CardsController
app/controllers/tools/chats/messages_controller.rb → Tools::Chats::MessagesController
```

### RESTful Controllers (Strict)

**Only standard CRUD actions (index, show, new, create, edit, update, destroy).** Non-RESTful actions are forbidden:

```ruby
# BAD - god controller with custom actions
class BoardsController
  def create_column
  def update_column
  def reorder_cards
end

# GOOD - separate controllers for each resource
class BoardColumnsController  # index, create, update, destroy
class BoardCardsController    # show, create, update, destroy
class BoardCardPositionsController  # update (for reordering)
```

State changes get dedicated controllers:
```ruby
# Instead of: patch :toggle_read, patch :toggle_starred
class EmailReadsController     # create/destroy (mark read/unread)
class EmailStarsController     # create/destroy (star/unstar)
class EmailArchivesController  # create/destroy (archive/unarchive)
class EmailTrashesController   # create/destroy (trash/restore)
```

### Routes

`scope module:` for namespacing without URL bloat:
```ruby
resources :tools do
  scope module: :tools do
    resource :board, only: :show do
      scope module: :boards do
        resources :columns
        resources :cards
      end
    end
  end
end

# Top-level resources for simpler URLs (IDs are globally unique)
resources :cards do
  scope module: :cards do
    resources :comments
    resources :attachments
  end
end
```

### Real-time (ActionCable)

- **ChatChannel** — messaging, typing indicators, presence
- **DocumentChannel** — collaborative editing broadcasts
- **NotificationChannel** — per-user stream (`notifications:#{user.id}`) for real-time notification delivery
- Connection authenticates via signed session cookie


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smgdkngt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
