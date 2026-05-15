---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Technology Stack

- **Backend**: Ruby on Rails 8.1.1, Ruby 3.4.7
- **Database**: PostgreSQL with `pg_search` for full-text search
- **Search**: Elasticsearch 8.x
- **Frontend**: Hotwire (Turbo, Stimulus), React 16.9, Bootstrap 5.3
- **Asset Pipeline**: esbuild for JavaScript, Sass for CSS
- **Background Jobs**: Resque with Redis, Solid Queue
- **Authentication**: Devise
- **PDF Generation**: Grover (Puppeteer-based, uses Chromium in Docker)
- **Testing**: RSpec
- **Containerization**: Docker, Docker Compose

## Docker Architecture

This project runs entirely in Docker containers. All commands must be executed inside containers using `docker compose run --rm`.

### Docker Services

- **db**: PostgreSQL 17.6 on port 5432
- **redis**: Redis 7 on port 6379
- **rails**: Main Rails application on port 3000
- **resque**: Background job workers
- **css**: CSS asset watcher
- **js**: JavaScript asset builder
- **test**: Test runner

### Docker Image

- Base image: `learningtapestry/lcms-core:dev` (built from `Dockerfile.dev`)
- Includes Ruby 3.4.7, Node.js 22, Yarn, PostgreSQL client, Chromium for PDF generation
- Uses volumes: `bundle`, `postgres-17.6`, `redis-7`

## Development Commands

All commands run inside disposable Docker containers with `--rm` flag.

### Setup

```bash
# Build the Docker image
docker build -f Dockerfile.dev -t learningtapestry/lcms-core:dev .

# Start all services
docker compose up

# Install dependencies
docker compose run --rm rails bundle install
docker compose run --rm rails yarn install

# Database setup
docker compose run --rm rails rails db:create
docker compose run --rm rails rails db:migrate
docker compose run --rm rails rails db:seed
```

### Development

```bash
# Start Rails server
docker compose up rails

# Start all services (Rails, Resque, CSS/JS watchers)
docker compose up

# Start specific services
docker compose up rails resque

# Rails console
docker compose run --rm rails rails console

# Rails commands
docker compose run --rm rails rails routes
docker compose run --rm rails rails db:migrate
docker compose run --rm rails rails db:rollback
```

### Asset Compilation

```bash
# Build JavaScript
docker compose run --rm js yarn build

# Build CSS once
docker compose run --rm rails yarn build:css

# Watch CSS for changes
docker compose up css
```

### Testing

**IMPORTANT**: The `.rspec` file contains a custom `--pattern` that includes plugin specs. When running individual spec files, you MUST override this pattern to avoid "No examples found" errors.

```bash
# Run all tests (uses default pattern from .rspec)
docker compose run --rm test bundle exec rspec

# Run specific test file (MUST override --pattern with the file path)
docker compose run --rm test bundle exec rspec --pattern 'spec/path/to/file_spec.rb' spec/path/to/file_spec.rb

# Run specific test by line (MUST override --pattern with the file path)
docker compose run --rm test bundle exec rspec --pattern 'spec/path/to/file_spec.rb' spec/path/to/file_spec.rb:42

# Setup test database
docker compose run --rm -e RAILS_ENV=test rails rails db:create
docker compose run --rm -e RAILS_ENV=test rails rails db:migrate
```

### Code Quality

```bash
# Run Rubocop
docker compose run --rm rails bundle exec rubocop

# Auto-fix style issues
docker compose run --rm rails bundle exec rubocop -a

# Security scans
docker compose run --rm rails bundle exec brakeman
docker compose run --rm rails bundle exec bundler-audit

# Run all pre-commit checks (Rubocop, Brakeman, YAML syntax, etc.)
docker compose run --rm rails overcommit --run

# Run pre-push checks (RSpec tests)
docker compose run --rm rails overcommit --run pre_push
```

### Git Hooks

Git hooks run checks automatically before commit and push. Since all tools run inside Docker, install the hooks that delegate to Docker containers:

```bash
# Install pre-commit and pre-push hooks
ln -sf ../../script/hooks/pre-commit .git/hooks/pre-commit
ln -sf ../../script/hooks/pre-push .git/hooks/pre-push
```

The hooks will run:
- **pre-commit**: Rubocop, Brakeman, ShellCheck, YAML syntax, trailing whitespace checks
- **pre-push**: RSpec tests

If Brakeman check fails, run interactive mode to review warnings:
```bash
docker compose run --rm -it rails bundle exec brakeman -I
```


### Background Jobs

```bash
# Start Resque workers (via docker-compose)
docker compose up resque

# Manual Resque worker
docker compose run --rm rails env QUEUE=* bundle exec rake resque:work

# Resque scheduler
docker compose run --rm rails bundle exec rake resque:scheduler
```

### Utility Commands

```bash
# Shell access to Rails container
docker compose run --rm rails bash

# Check Ruby version
docker compose run --rm rails ruby --version

# Check syntax of Ruby files
docker compose run --rm rails ruby -c app/helpers/some_helper.rb

# Database console
docker compose run --rm rails rails dbconsole
```

## Application Architecture

### Core Domain Models

**Documents and Materials**
- `Document`: Lesson documents imported from Google Docs, with hierarchical curriculum structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [learningtapestry/lcms-core](https://github.com/learningtapestry/lcms-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
