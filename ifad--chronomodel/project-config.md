---
trigger: always_on
description: ChronoModel is a Ruby gem that provides temporal database functionality (Type-2 Slowly-Changing Dimensions) for PostgreSQL using Active Record. It implements Oracle-style "Flashback Queries" using updatable views, table inheritance, and INSTEAD OF triggers.
---

# ChronoModel - Temporal Database Extensions for PostgreSQL

ChronoModel is a Ruby gem that provides temporal database functionality (Type-2 Slowly-Changing Dimensions) for PostgreSQL using Active Record. It implements Oracle-style "Flashback Queries" using updatable views, table inheritance, and INSTEAD OF triggers.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Environment Setup
- **Ruby Version**: Requires Ruby >= 3.0 (Ruby 3.2.3 validated)
- **Rails Support**: Active Record >= 7.0 (supports Rails 7.0, 7.1, 7.2, 8.0, and edge)
- **PostgreSQL**: Requires PostgreSQL >= 9.4 (PostgreSQL 16.9 validated)
- **Required Extension**: `btree_gist` PostgreSQL extension must be installed

### Bootstrap the Development Environment
```bash
# Install system dependencies
sudo gem install bundler

# Configure bundler for local installation
bundle config set --local path 'vendor/bundle'

# Install dependencies - takes 3-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
bundle install

# Start PostgreSQL service
sudo service postgresql start

# Create superuser for development
sudo -u postgres createuser -s $USER

# Create development database
createdb chronomodel

# Install required PostgreSQL extension
psql -d chronomodel -c "CREATE EXTENSION IF NOT EXISTS btree_gist;"

# Copy test configuration
cp spec/config.yml.example spec/config.yml
# Edit spec/config.yml with your database credentials:
# adapter: chronomodel
# host: localhost  
# username: [your_username]
# password: [your_password]
# database: chronomodel
# pool: 11
```

### Build and Test
```bash
# Build the gem - takes <1 second
bundle exec gem build chrono_model.gemspec

# Install locally - takes <1 second  
bundle exec gem install --local chrono_model-4.0.0.gem

# Run linting - takes 2-3 seconds. NEVER CANCEL.
bundle exec rubocop lib

# Run core tests - requires database setup. NEVER CANCEL. Set timeout to 30+ minutes.
bundle exec rspec spec/chrono_model

# Run full test suite including integration - NEVER CANCEL. Set timeout to 60+ minutes.
bundle exec rake

# Create test Rails application for integration testing - takes 30+ seconds
bundle exec rake testapp:create
```

### Multi-Rails Testing
ChronoModel supports multiple Rails versions using Appraisal:
```bash
# Test against specific Rails version - each takes 30+ minutes. NEVER CANCEL.
BUNDLE_GEMFILE=gemfiles/rails_7.0.gemfile bundle exec rspec
BUNDLE_GEMFILE=gemfiles/rails_7.1.gemfile bundle exec rspec  
BUNDLE_GEMFILE=gemfiles/rails_7.2.gemfile bundle exec rspec
BUNDLE_GEMFILE=gemfiles/rails_8.0.gemfile bundle exec rspec
BUNDLE_GEMFILE=gemfiles/rails_edge.gemfile bundle exec rspec
```

## Validation

### Database Setup Validation
Always validate the database setup before running tests:
```bash
# Test basic connection
psql -d chronomodel -c "SELECT version();"

# Verify btree_gist extension
psql -d chronomodel -c "SELECT * FROM pg_extension WHERE extname = 'btree_gist';"
```

### Code Quality
```bash
# Always run linting before committing - takes 2-3 seconds
bundle exec rubocop

# Check all Ruby files for style issues
bundle exec rubocop lib spec

# Auto-fix simple style issues  
bundle exec rubocop --autocorrect
```

### Commit Message Guidelines
Follow the project's commit message standards as outlined in [CONTRIBUTING.md](../CONTRIBUTING.md):

- **Reference**: Follow [How to Write a Git Commit Message](https://cbea.ms/git-commit/#seven-rules)
- **Format**: Use imperative mood ("Add feature" not "Added feature")
- **Length**: Limit subject line to 50 characters, body lines to 72 characters
- **Structure**: 
  ```
  Short summary (50 chars max)
  
  Detailed explanation if needed (72 chars per line)
  
  - Use bullet points for multiple changes
  - Reference issues with "Fixes #123" or "Closes #456"
  ```
- **Examples**:
  ```
  Fix temporal query performance regression
  
  Add support for Rails 8.0 compatibility
  
  Update dependencies for security patches
  
  Fixes #123
  ```
- **Best Practices**:
  - Keep commits atomic (one logical change per commit)
  - Write clear, descriptive commit messages
  - Reference related issues and pull requests
  - Avoid generic messages like "Fix bug" or "Update code"

### Testing Strategy
- **Core Tests**: Run `bundle exec rspec spec/chrono_model` for basic functionality
- **Integration Tests**: Run `bundle exec rake` for full test suite including Rails integration
- **Multi-Version**: Use gemfiles to test against different Rails versions
- **CI Validation**: Tests run against Ruby 3.0-3.4 and Rails 7.0-8.0 in CI

## Common Tasks

### Repository Structure
```
/home/runner/work/chronomodel/chronomodel/
├── lib/chrono_model/           # Main gem code (29 Ruby files, ~2,500 lines)
│   ├── adapter/               # Database adapter customizations
│   ├── patches/               # Active Record patches
│   ├── time_machine/          # Temporal query functionality
│   ├── adapter.rb             # Main adapter class

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ifad/chronomodel](https://github.com/ifad/chronomodel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
