---
trigger: always_on
description: A Redmine plugin that enables Git hosting through Gitolite, providing repository management, SSH key handling, and Git Smart-HTTP functionality.
---

# Redmine Git Hosting Plugin

A Redmine plugin that enables Git hosting through Gitolite, providing repository management, SSH key handling, and Git Smart-HTTP functionality.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Critical: This is a Redmine Plugin, NOT a Standalone Application

**IMPORTANT**: This codebase is a Redmine plugin that requires a full Redmine installation to run. You cannot run this plugin independently. The plugin extends Redmine's functionality but must be installed within an existing Redmine application.

## Working Effectively

### Development Setup (Linting and Code Quality Only)
- Bootstrap development environment:
  - `sudo gem install bundler`
  - `touch .enable_dev` (enables development gems)
  - `bundle config set --local path 'vendor/bundle'`
  - `bundle install --jobs 4 --retry 3` -- takes 80 seconds. Set timeout to 120+ seconds.
  - `yarn install` -- takes 30 seconds. Set timeout to 60+ seconds.

### Linting and Code Quality (NEVER CANCEL - All Required for CI)
- `bundle exec rubocop -S` -- Ruby code linting, takes 2 seconds. NEVER CANCEL.
- `bundle exec slim-lint app/views` -- View template linting, takes 3 seconds. NEVER CANCEL.
- `node_modules/.bin/stylelint "assets/stylesheets/*.css"` -- CSS linting, takes 1 second. NEVER CANCEL.
- `bundle exec brakeman` -- Security scanning, takes 3 seconds. NEVER CANCEL.

### What You CANNOT Do (Plugin Limitations)
- **DO NOT** try to run this as a standalone Rails application
- **DO NOT** attempt `rails server` or similar - there is no main application
- **DO NOT** run `rake` commands directly - they require Redmine context
- **DO NOT** try to run RSpec tests without full Redmine setup
- **DO NOT** expect database migrations to work without Redmine

### Full Plugin Testing (Requires Redmine Installation)
**WARNING**: The following requires a complete Redmine setup and takes 45+ minutes:

Complete setup process (from .github/workflows/test.yml):
1. Install system dependencies:
   ```bash
   sudo apt-get update --yes --quiet
   sudo apt-get install --yes --quiet build-essential cmake libgpg-error-dev libicu-dev libpq-dev libmysqlclient-dev libssh2-1 libssh2-1-dev pkg-config subversion
   ```

2. Setup Redmine (clone redmine/redmine repository):
   ```bash
   git clone https://github.com/redmine/redmine.git /path/to/redmine
   cd /path/to/redmine
   ```

3. Install plugin and dependencies:
   ```bash
   # Clone this plugin into redmine/plugins/redmine_git_hosting
   # Clone required plugins:
   git clone https://github.com/AlphaNodes/additionals.git plugins/additionals
   git clone https://github.com/dosyfier/redmine_sidekiq.git plugins/redmine_sidekiq -b fix-rails-6
   ```

4. Setup Gitolite:
   ```bash
   # Install and configure Gitolite (see CI workflow for complete steps)
   ssh-keygen -N '' -f plugins/redmine_git_hosting/ssh_keys/redmine_gitolite_admin_id_rsa
   # Full Gitolite setup required...
   ```

5. Run tests (45+ minutes total):
   ```bash
   bundle exec rake redmine_git_hosting:ci:all
   ```
   **NEVER CANCEL** - Tests take 45+ minutes. Set timeout to 60+ minutes.

## Validation

### Always Run These Before Committing
- `bundle exec rubocop -S` -- fixes most style issues automatically
- `bundle exec slim-lint app/views` -- checks view templates
- `node_modules/.bin/stylelint "assets/stylesheets/*.css"` -- checks CSS
- `bundle exec brakeman` -- security scanning

### Validation Scenarios (Requires Full Redmine Setup)
Since this is a plugin, validation requires:
1. Full Redmine installation with this plugin
2. Gitolite server setup with SSH keys
3. Database configured (PostgreSQL or MySQL)
4. Test scenarios include:
   - Repository creation and management
   - SSH key management
   - Git Smart-HTTP functionality
   - Mirror and deployment credential management

## Common Tasks

### Repo Structure (ls -la output)
```
total 248
drwxr-xr-x  15 runner docker  4096 Aug 11 11:52 .
drwxr-xr-x   3 runner docker  4096 Aug 11 11:38 ..
drwxr-xr-x   2 runner docker  4096 Aug 11 11:41 .bundle
-rw-r--r--   1 runner docker     0 Aug 11 11:49 .enable_dev
drwxr-xr-x   7 runner docker  4096 Aug 11 11:47 .git
drwxr-xr-x   3 runner docker  4096 Aug 11 11:49 .github
-rw-r--r--   1 runner docker   183 Aug 11 11:38 .gitignore
-rw-r--r--   1 runner docker  2121 Aug 11 11:38 .rubocop.yml
-rw-r--r--   1 runner docker  8299 Aug 11 11:38 .rubocop_todo.yml
-rw-r--r--   1 runner docker  1270 Aug 11 11:38 .slim-lint.yml
-rw-r--r--   1 runner docker  5635 Aug 11 11:38 .stylelintrc.json
-rw-r--r--   1 runner docker  2478 Aug 11 11:38 AUTHORS
-rw-r--r--   1 runner docker 36496 Aug 11 11:38 CHANGELOG.md
-rw-r--r--   1 runner docker  1573 Aug 11 11:38 Gemfile
-rw-r--r--   1 runner docker  9468 Aug 11 11:51 Gemfile.lock
-rw-r--r--   1 runner docker  1046 Aug 11 11:38 LICENSE
-rw-r--r--   1 runner docker  1527 Aug 11 11:38 README.md
drwxr-xr-x  13 runner docker  4096 Aug 11 11:38 app
drwxr-xr-x   5 runner docker  4096 Aug 11 11:38 assets
drwxr-xr-x   3 runner docker  4096 Aug 11 11:38 config
drwxr-xr-x   5 runner docker  4096 Aug 11 11:38 contrib

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redmine-git-hosting/redmine_git_hosting](https://github.com/redmine-git-hosting/redmine_git_hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
