---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hong₿ao is a Ruby on Rails 8 application for generating Bitcoin paper wallets in red envelope format. It uses Hotwire (Turbo + Stimulus) for frontend interactivity and ViewComponents for UI components.

## Key Commands

### Development
```bash
bin/dev              # Start development server on port 3001
bin/rails db:setup   # Setup database with schema
bin/rails db:seed    # Seed with sample data
bin/rails console    # Rails console
```

### Testing
```bash
bin/rspec                                    # Run all RSpec tests
bin/rspec spec/models/paper_spec.rb         # Run specific test file
bin/e2e                                      # Run E2E tests (4 parallel workers)
bin/e2e --parallel[8]                        # Run E2E tests with 8 workers
bin/e2e e2e/playwright/e2e/visual_editor.spec.js  # Run specific E2E test

# For faster single file testing:
cd e2e && npx playwright test calendar.spec.js  # Run single test file directly
```

### Linting & Security
```bash
bundle exec rubocop                          # Run Ruby linter
bundle exec rubocop -a                       # Auto-fix Ruby linting issues
bundle exec brakeman                         # Run security scanner
yarn audit                                   # Check JavaScript vulnerabilities
```

### Asset Building
```bash
yarn build          # Build all assets
yarn watch:js       # Watch and rebuild JavaScript
yarn watch:css      # Watch and rebuild CSS
```

## Architecture & Structure

### Core Models
- **Paper**: Main model for Bitcoin paper wallets, handles wallet generation and rendering
- **User**: User accounts with authentication via Devise
- **Bundle**: Collection of papers for bulk operations
- **Token**: Authentication tokens for various purposes
- **Input**: Tracks Bitcoin inputs for funding wallets

### Frontend Architecture
- **ViewComponents** in `app/components/` for reusable UI (e.g., `Papers::LineItemComponent`)
- **Stimulus Controllers** in `app/javascript/controllers/` for client-side behavior
- **Turbo Streams** for real-time updates without page reloads
- **Tailwind CSS v4** for styling

### Key Features Implementation
- **Bitcoin Functionality**: Uses `bitcoin-ruby` gem with offline wallet generation capability
- **AI Design Generation**: Integrated via `ruby_llm` gem for custom wallet designs
- **Admin Interface**: ActiveAdmin at `/admin` route
- **Payment Processing**: Stripe integration for payments
- **Background Jobs**: SolidQueue for async processing

### Testing Approach
- **RSpec** for unit/integration tests with FactoryBot for test data
- **Playwright** for E2E browser testing across Chrome, Firefox, Safari
- **VCR** for recording HTTP interactions in tests
- **Parallel Tests** supported for faster test runs

### Database
- SQLite for all environments
- Separate databases for cache (SolidCache), queue (SolidQueue), and cable (SolidCable)
- Schema tracked in `db/schema.rb`

### Important Patterns
- Use ViewComponents for new UI components instead of partials
- Turbo Streams via model callbacks (follow Paper model pattern)
- Component-based broadcasting for real-time updates
- Stimulus controllers for JavaScript behavior
- Admin resources defined in `app/admin/`
- Background jobs inherit from `ApplicationJob` and only update models (broadcasting happens automatically)

### Development Notes
- **Port 3001** is used for development server (not 3000)
- Credentials managed via Rails credentials (requires `master.key`)
- Submodules present - run `git submodule update --init --recursive` after cloning
- Environment variables via `.env` file
- Letter Opener used for email testing in development

## Hongbao Rails Development with ClaudeOnRails

This project uses ClaudeOnRails to create an intelligent swarm of AI agents specialized in different aspects of Rails development.

### How to Use

Simply describe what you want to build or fix, and the swarm will automatically coordinate the implementation:

```bash
# Start the swarm
claude-swarm orchestrate

# Then just describe your task
claude "Add user authentication with email confirmation"
claude "Optimize the dashboard queries that are running slowly"
claude "Create an API endpoint for mobile app integration"
```

### Swarm Architecture

The following specialized agents work together to implement your requests:

- **Architect**: Coordinates all development and makes high-level decisions
- **Models**: Handles ActiveRecord models, migrations, and database design
- **Controllers**: Manages request handling, routing, and controller logic
- **Views**: Creates and maintains views, View Components, layouts, and partials
- **Stimulus**: Implements interactive features with Stimulus controllers
- **Services**: Extracts business logic into service objects
- **Jobs**: Handles background processing and async tasks
- **Tests**: Ensures comprehensive test coverage with RSpec
- **DevOps**: Manages deployment and production configurations

## Project Conventions

### Code Style
- Follow Rails conventions and best practices
- Use RuboCop for Ruby style enforcement
- Prefer clarity over cleverness

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbounmy/hongbao_bitcoin](https://github.com/sbounmy/hongbao_bitcoin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
