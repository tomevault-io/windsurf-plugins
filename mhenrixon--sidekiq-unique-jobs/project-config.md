---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

sidekiq-unique-jobs is a Sidekiq middleware gem that prevents duplicate jobs from being enqueued or executed. It provides sophisticated locking mechanisms using Redis to ensure job uniqueness based on configurable parameters.

## Development Commands

### Testing
```bash
# Run all tests
bundle exec rspec

# Run specific test file
bundle exec rspec spec/path/to/spec.rb

# Run specific test by line number
bundle exec rspec spec/path/to/spec.rb:42

# Run tests with specific appraisals (different Sidekiq versions)
bundle exec appraisal rspec
bundle exec appraisal sidekiq-6.0 rspec
```

### Code Quality
```bash
# Run rubocop linter
bundle exec rake rubocop

# Run reek (code smell detector)
bundle exec rake reek

# Run all style checks
bundle exec rake style

# Generate documentation
bundle exec rake yard
```

### Build and Release
```bash
# Run all checks (style, tests, documentation)
bundle exec rake

# Release a new gem version (only for maintainers)
bundle exec rake release
```

## Architecture

### Lock Types

The gem implements multiple lock strategies that control when and how uniqueness is enforced:

1. **Client-side locks** (prevent duplicate enqueuing):
   - `until_executing` - Lock from push until job starts executing
   - `until_executed` - Lock from push until job completes execution
   - `until_expired` - Lock from push until configured TTL expires
   - `until_and_while_executing` - Combination lock (both phases)

2. **Server-side locks** (prevent duplicate execution):
   - `while_executing` - Lock only during job execution
   - `while_executing_reject` - Same as above but rejects conflicts

Lock implementations inherit from `SidekiqUniqueJobs::Lock::BaseLock` (lib/sidekiq_unique_jobs/lock/base_lock.rb) and implement two key methods:
- `lock` - Attempt to acquire the lock
- `execute` - Execute the job with appropriate lock behavior

### Core Components

**Middleware Stack:**
- `SidekiqUniqueJobs::Middleware::Client` - Intercepts job enqueuing
- `SidekiqUniqueJobs::Middleware::Server` - Intercepts job execution
- These must be manually configured in Sidekiq initializer (not auto-loaded)

**Lock Management:**
- `Locksmith` (lib/sidekiq_unique_jobs/locksmith.rb) - Central lock manager that interfaces with Redis
- `LockDigest` - Generates unique digest from job parameters (queue, class, args)
- `LockConfig` - Extracts and normalizes lock configuration from job options
- `Key` - Manages Redis key naming with configurable prefixes

**Conflict Resolution:**
- Strategies in `lib/sidekiq_unique_jobs/on_conflict/` handle lock conflicts:
  - `log` - Log the conflict
  - `raise` - Raise exception (for retry)
  - `reject` - Send to dead queue
  - `replace` - Delete existing job and retry
  - `reschedule` - Delay and retry
- Inherit from `SidekiqUniqueJobs::OnConflict::Strategy`

**Lua Scripts:**
- All Redis operations use Lua scripts in `lib/sidekiq_unique_jobs/lua/`
- This ensures atomic operations and consistency
- Scripts are loaded and executed via `Script::Caller` mixin
- Template system with shared functions in `lua/shared/`

**Orphan Cleanup:**
- `SidekiqUniqueJobs::Orphans::Manager` - Coordinates reaper lifecycle
- `SidekiqUniqueJobs::Orphans::RubyReaper` - Ruby-based cleanup (default)
- `SidekiqUniqueJobs::Orphans::LuaReaper` - Lua-based cleanup (faster but locks Redis)
- Reapers run periodically to clean up stale locks from crashed processes

### Configuration System

Global configuration via `SidekiqUniqueJobs.configure` block:
- Uses `Concurrent::MutableStruct` for thread-safe config
- Defined in `lib/sidekiq_unique_jobs/config.rb`
- Supports custom locks and strategies via `add_lock` and `add_strategy`

Per-worker configuration via `sidekiq_options`:
- `lock` - Lock type (required)
- `on_conflict` - Conflict strategy (can differ for client/server)
- `lock_timeout` - How long to wait for lock acquisition
- `lock_ttl` - Lock expiration time
- `lock_args_method` - Custom method/proc to filter uniqueness args
- `unique_across_queues` - Ignore queue in digest calculation
- `unique_across_workers` - Ignore worker class in digest calculation

### Redis Integration

The gem uses `redis-client` (not `redis` gem) via Sidekiq's connection pool. Wrapper classes in `lib/sidekiq_unique_jobs/redis/` provide object-oriented interfaces:
- `Redis::String` - String operations
- `Redis::Hash` - Hash operations
- `Redis::Set` - Set operations
- `Redis::SortedSet` - Sorted set operations
- `Redis::List` - List operations

All inherit from `Redis::Entity` base class.

### Reflection System

The gem provides hooks for observability (metrics, logging) via `SidekiqUniqueJobs.reflect`:
- `locked` - Lock acquired successfully
- `lock_failed` - Could not acquire lock
- `unlocked` - Lock released
- `unlock_failed` - Lock release failed
- `timeout` - Lock acquisition timed out
- `execution_failed` - Job execution raised error
- Other reflection points defined in `lib/sidekiq_unique_jobs/reflections.rb`

## Testing Guidelines

- Integration tests in `spec/sidekiq_unique_jobs/lock/*_spec.rb` cover each lock type end-to-end

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhenrixon/sidekiq-unique-jobs](https://github.com/mhenrixon/sidekiq-unique-jobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
