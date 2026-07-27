---
trigger: always_on
description: validates :email, presence: true
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Quick Reference

### Template Markers
When using this as a template for new projects, look for these markers:
- `[PROJECT-SPECIFIC]` - Must be updated for each project
- `[OPTIONAL]` - Remove section if not applicable to your project
- `[CUSTOMIZE]` - Adapt values/examples to project needs

### Essential Commands Quick Reference
| Task | Command |
|------|---------|
| Run all tests | `bundle exec rspec` |
| Run linter | `bundle exec rubocop` |
| Auto-fix lint | `bundle exec rubocop -a` |
| Start server | `bin/dev` |
| Console | `bin/rails console` |
| DB migrate | `bin/rails db:migrate` |
| Security scan | `bundle exec rake appfactory:test:brakeman` |

---

## Code Style & Project Guidelines

- **Core Language:** Always use **Ruby**.
- **Framework:** Rails 8.0.2 (per Gemfile) `[PROJECT-SPECIFIC]`
- **Ruby Version:** Ruby 3.4.3 (per .ruby-version) `[PROJECT-SPECIFIC]`
- **Mental Model:** Think like Sandy Metz when she wrote Practical Object-Oriented Design in Ruby.
- **Code Style:** Apply the style, conventions, and rules from `.rubocop.yml` and all config files in `config/rubocop/` (base.yml, rails.yml, rspec.yml, metrics.yml, layout.yml, obsession.yml). **IMPORTANT: All generated code MUST pass RuboCop checks without violations.**
- **Gems & Dependencies:** Review the `Gemfile` and `Gemfile.lock` when providing code examples. All listed gems are available.
- **Testing Plugins:** This project uses: `[CUSTOMIZE]`
  - rubocop-rspec
  - rubocop-performance
  - rubocop-rails
  - rubocop-factory_bot
  - rubocop-rspec_rails
  - rubocop-obsession
- **Temporary Files:** Always use the project's `./tmp` directory for temporary files.
- **Documentation Files:** All explanation, documentation, and reference markdown files (*.md) should be written to the `./tmp` directory, NOT to the project root or other directories, unless told otherwise.

---

## Essential Commands

### Development
```bash
bin/setup              # Initial setup: installs dependencies, prepares database, starts dev server
bin/dev                # Start Rails development server
bin/rails server       # Start Rails server directly
bin/rails console      # Open Rails console
```

### Testing
```bash
bundle exec rspec                                  # Run all tests
bundle exec rspec spec/services/                   # Run specific directory
bundle exec rspec spec/services/some_spec.rb      # Run specific file
bundle exec rspec spec/services/some_spec.rb:42   # Run specific line

bundle exec rake appfactory:test:simplecov         # Run tests with coverage [PROJECT-SPECIFIC]
bundle exec rake appfactory:test:rubocop           # Run RuboCop checks [PROJECT-SPECIFIC]
bundle exec rake appfactory:test:brakeman          # Run security analysis [PROJECT-SPECIFIC]
```

### Database
```bash
bin/rails db:prepare                               # Setup database
bin/rails db:migrate                               # Run migrations
bundle exec rake appfactory:database:reset         # Reset database [PROJECT-SPECIFIC]
bundle exec rake appfactory:database:recreate[development]  # Drop and recreate [PROJECT-SPECIFIC]
```

### Code Quality
```bash
bundle exec rubocop                                # Run all RuboCop checks
bundle exec rubocop -a                             # Auto-fix RuboCop violations
bundle exec rubocop spec/services/some_spec.rb    # Check specific file
```

---

## Project Directory Structure

```
app/
├── assets/              # Static assets (stylesheets, images)
├── builders/            # Builder pattern for complex object construction [OPTIONAL]
├── controllers/         # HTTP request handlers (keep thin!)
├── forms/               # Form objects for complex multi-model forms
├── helpers/             # View helpers (use sparingly)
├── inputs/              # Input objects for request parameter parsing [OPTIONAL]
├── jobs/                # Background jobs (Solid Queue)
├── lib/                 # App-dependent library code (e.g., Rodauth config)
├── mailers/             # ActionMailer classes (prefer service objects)
├── models/              # ActiveRecord models (data + associations only)
├── policies/            # Pundit authorization policies
├── presenters/          # View presenters/decorators [OPTIONAL]
├── queries/             # Query objects for complex database queries
├── services/            # ServiceWrapper business logic (primary location)
├── validators/          # Custom ActiveModel validators
├── views/               # Slim templates
└── workers/             # Legacy background workers [OPTIONAL]

config/
├── environments/        # Environment-specific configuration
├── initializers/        # Boot-time initialization
├── locales/             # I18n translation files
├── rubocop/             # Modular RuboCop configuration
└── settings/            # Application settings (config gem)

lib/
├── core_extensions/     # Core Ruby class extensions
├── monkey_patches/      # Runtime modifications (use sparingly)
├── modules/             # Reusable utility modules
├── tasks/               # Custom Rake tasks
└── utilities/           # Helper utilities

spec/
├── factories/           # FactoryBot test data definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrishough/my-configurations](https://github.com/chrishough/my-configurations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
