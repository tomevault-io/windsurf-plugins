---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## PromptEngine Rails Engine

PromptEngine is a mountable Rails engine for AI prompt management. It provides a centralized admin
interface where teams can create, version, test, and optimize their AI prompts without deploying
code changes.

## Development Commands

### Initial Setup

```bash
bundle install
bundle exec rake setup                      # Complete setup for development and testing
```

This setup task:
- Removes existing migrations and databases
- Installs engine migrations into dummy app
- Creates and migrates both development and test databases
- Seeds development database with sample data

### Running the Development Server

```bash
cd spec/dummy && rails s
```

Access the admin interface at `http://localhost:3000/prompt_engine`

### Manual Database Setup (if needed)

```bash
cd spec/dummy
bundle exec rails prompt_engine:install:migrations  # REQUIRED: Install engine migrations first
bundle exec rails db:create db:migrate db:seed      # Setup development database
RAILS_ENV=test bundle exec rails db:create db:migrate  # Setup test database
```

**Note**: Engine migrations are NOT automatically loaded. You must explicitly run `rails prompt_engine:install:migrations` before running `db:migrate`.

### Running Tests

```bash
bundle exec rspec                           # Run all tests with coverage
bundle exec rspec spec/models               # Run model tests
bundle exec rspec spec/requests             # Run request specs (controllers)
bundle exec rspec spec/system               # Run system tests
bundle exec rspec path/to/spec.rb           # Run specific test file
bundle exec rspec path/to/spec.rb:42        # Run specific test by line number
bundle exec rspec --format documentation    # Run with detailed output
bundle exec rspec --fail-fast              # Stop on first failure

# Coverage report
open coverage/index.html                    # View SimpleCov coverage report
```

### Linting

```bash
bundle exec rubocop                         # Run RuboCop with Rails Omakase style
bundle exec rubocop -a                      # Auto-correct offenses
```

### Rake Tasks

```bash
bundle exec rake setup                      # Setup dummy app for development
bundle exec rake spec                       # Run all specs
bundle exec rake prompt_engine:setup        # Alternative setup command
bin/rails prompt_engine:install:migrations  # Install engine migrations in host app
```

## Architecture Overview

### Core Models & Relationships

**PromptEngine::Prompt**

- Central model containing prompt templates with `{{variable}}` syntax
- Has many versions (auto-versioned on content/system_message changes)
- Has many parameters (auto-detected from variables)
- Status enum: draft, active, archived
- Key method: `render(variables: {})` for template rendering

**PromptEngine::PromptVersion**

- Immutable snapshots of prompts
- Auto-incremented version numbers
- Stores content, system_message, model_config at time of creation
- Key method: `restore!` to revert prompt to this version

**PromptEngine::Parameter**

- Defines expected variables with types and validation
- Types: string, integer, decimal, boolean, datetime, date, array, json
- Auto-generates appropriate form inputs

### Service Objects

**VariableDetector**

- Extracts `{{variables}}` from prompt content
- Infers types from variable names (e.g., `_at` → datetime)
- Validates provided variables against template

**PlaygroundExecutor**

- Handles AI provider communication (Anthropic, OpenAI)
- Manages API keys from Rails credentials
- Formats requests and parses responses

### Testing Philosophy

Read `.ai/RSPEC-TESTS.md` before writing tests. Key principles:

- Use request specs instead of controller specs for testing controllers
- Test full request/response cycle
- Use FactoryBot for test data creation
- Keep unit tests focused on single behaviors
- Test both happy and unhappy paths
- One expectation per unit test (multiple expectations OK for integration tests)
- Test edge cases and validations
- Use `let` for lazy-loading test data
- Organize with `describe` and `context` blocks

### CSS Architecture

- BEM methodology with `.component__element--modifier` pattern
- Foundation based on shadcn/ui aesthetic (without React)
- Propshaft for asset management
- Components organized in separate files (buttons, forms, tables, etc.)
- All styles must be explicitly imported in `application.css`
- Component prefix: `ap-` for all PromptEngine components
- Color variables defined in `foundation.css`
- Spacing system based on 4px units

## Dependencies

- **Rails**: 8.0.2+ (mountable engine)
- **Ruby**: 3.0+
- **ruby_llm**: For AI provider integration
- **bcrypt**: For encryption support
- **RSpec Rails**: Testing framework
- **FactoryBot**: Test data generation
- **VCR**: External API testing
- **Capybara**: System testing
- **SimpleCov**: Code coverage
- **Rubocop Rails Omakase**: Code style enforcement

## Key Implementation Details

### Version Control System

- Automatic versioning tracks changes to content, system_message, and model settings
- Version history preserved with change descriptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aviflombaum/prompt_engine](https://github.com/aviflombaum/prompt_engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
