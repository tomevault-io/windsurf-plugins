---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClawDeck is a Rails 8.1 todo application with passwordless email authentication (6-digit verification codes via Resend). It's deployed to DigitalOcean VPS with automatic CI/CD via GitHub Actions.

## Development Commands

### Initial Setup
```bash
bin/setup              # Install dependencies, prepare database, start server
bin/setup --skip-server  # Setup without starting the server
bin/setup --reset      # Setup with database reset
```

### Running the Application
```bash
bin/dev                # Start development server (web + Tailwind CSS watch)
bin/rails server       # Start web server only
bin/rails tailwindcss:watch  # Watch and rebuild Tailwind CSS
```

### Database
```bash
bin/rails db:prepare   # Create, migrate, and seed database
bin/rails db:migrate   # Run pending migrations
bin/rails db:reset     # Drop, create, migrate, seed
bin/rails db:seed:replant  # Truncate and reseed
```

### Testing
```bash
bin/rails test         # Run all unit/integration tests
bin/rails test:system  # Run system tests (Capybara + Selenium)
bin/rails test test/models/user_test.rb  # Run specific test file
bin/rails test test/models/user_test.rb:10  # Run specific test line
```

### Code Quality and Security
```bash
bin/rubocop            # Run RuboCop linter (Omakase Ruby style)
bin/rubocop -a         # Auto-correct offenses
bin/brakeman           # Security analysis
bin/bundler-audit      # Check for vulnerable gem versions
bin/importmap audit    # Check for vulnerable JavaScript dependencies
bin/ci                 # Run full CI suite (setup, linting, security, tests)
```

### Asset Management
```bash
bin/rails assets:precompile  # Precompile assets for production
bin/importmap pin <package>  # Pin JavaScript package from CDN
bin/importmap unpin <package>  # Unpin JavaScript package
```

### Deployment
```bash
ssh root@YOUR_SERVER_IP  # SSH to production VPS
systemctl status puma         # Check Puma status
systemctl status solid_queue  # Check Solid Queue status
systemctl restart puma        # Restart web server
systemctl restart solid_queue # Restart background jobs
tail -f /var/log/clawdeck/puma.log  # View application logs
tail -f /var/log/clawdeck/solid_queue.log  # View job logs
```

## Architecture

### Technology Stack
- **Ruby/Rails**: 3.3.1 / 8.1.0
- **Database**: PostgreSQL with multi-database setup (primary, cache, queue, cable)
- **Background Jobs**: Solid Queue (database-backed)
- **Caching**: Solid Cache (database-backed)
- **WebSockets**: Solid Cable (database-backed)
- **Email**: Resend API (passwordless authentication with 6-digit codes)
- **Frontend**: Hotwire (Turbo + Stimulus), Tailwind CSS, importmap for JavaScript
- **Image Processing**: Active Storage with image_processing gem
- **Deployment**: DigitalOcean VPS + GitHub Actions auto-deploy
- **Web Server**: Puma with Nginx reverse proxy

### Application Structure
The application follows standard Rails 8 conventions with these key components:

- **Module Name**: `ClawDeck` (config/application.rb:9)
- **Solid Stack**: Uses solid_cache, solid_queue, and solid_cable instead of Redis
- **Hotwire-first**: Built for Turbo Drive navigation with minimal JavaScript
- **Asset Pipeline**: Propshaft for static assets, importmap-rails for JavaScript modules

### Key Models and Relationships
- **User**: Email-based authentication with 6-digit verification codes (15-minute expiry)
  - `has_many :sessions` - User sessions with user_agent and IP tracking
  - `has_many :projects` - User's todo projects
  - `has_many :tasks` - User's tasks
  - Creates welcome project with sample task lists on signup

- **Project**: Todo projects with optional image attachments
  - `belongs_to :user`
  - `has_many :tasks` (dependent: destroy)
  - `has_many :task_lists` (dependent: destroy)
  - `has_one_attached :image` - Project avatars (max 512KB, 256x256px, JPEG/PNG/WebP)
  - Position-based ordering with `default_scope`
  - Creates default "Tasks" task list on creation

- **TaskList**: Groupings of tasks within a project (Kanban-style columns)
  - `belongs_to :project`
  - `belongs_to :user`
  - `has_many :tasks` (dependent: destroy)
  - Position-based ordering with `default_scope`
  - Color options: gray, red, blue, lime, purple, yellow

- **Task**: Individual todo items with position management and completion tracking
  - `belongs_to :project`
  - `belongs_to :user`
  - `belongs_to :task_list`
  - Priority enum: none, low, medium, high
  - Position-based ordering (acts_as_list behavior without the gem)
  - Tracks `completed_at` and `original_position` for completion/restoration
  - Scopes: `incomplete` (ordered by position), `completed` (ordered by completed_at desc)

- **Session**: User sessions with device/IP tracking
  - `belongs_to :user`
  - Stored in signed, permanent, httponly, lax same-site cookies

### Authentication System
Passwordless email authentication using 6-digit verification codes:
1. User enters email address
2. System generates 6-digit code (valid for 15 minutes)
3. Code sent via Resend API
4. User verifies code to create session

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clawdeckio/clawdeck](https://github.com/clawdeckio/clawdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
