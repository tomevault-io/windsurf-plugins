---
trigger: always_on
description: InertiaCable is a Ruby gem + React npm package (`@inertia-cable/react`) that adds ActionCable broadcast integration to Inertia.js Rails apps. When a model changes, it broadcasts a lightweight JSON signal over WebSocket; the React client receives it and calls `router.reload()` to re-fetch props through Inertia's normal HTTP flow.
---

# CLAUDE.md — InertiaCable

## Project Overview

InertiaCable is a Ruby gem + React npm package (`@inertia-cable/react`) that adds ActionCable broadcast integration to Inertia.js Rails apps. When a model changes, it broadcasts a lightweight JSON signal over WebSocket; the React client receives it and calls `router.reload()` to re-fetch props through Inertia's normal HTTP flow.

**Key principle**: The controller remains the single source of truth for data — WebSockets carry only signals, not data payloads (except for explicit `broadcast_message_to` direct messages).

## Repository Structure

```
├── lib/                          # Ruby gem source
│   ├── inertia_cable.rb          # Main module entry point
│   └── inertia_cable/
│       ├── broadcastable.rb      # Core DSL mixed into ActiveRecord
│       ├── broadcast_job.rb      # ActiveJob for async broadcasts
│       ├── controller_helpers.rb # Controller `inertia_cable_stream` helper
│       ├── debounce.rb           # Cache-based server-side debounce
│       ├── suppressor.rb         # Thread-safe broadcast suppression
│       ├── engine.rb             # Rails engine (auto-includes modules)
│       ├── test_helper.rb        # Test assertions for broadcasts
│       ├── version.rb            # Version constant
│       └── streams/
│           └── stream_name.rb    # Stream name resolution & HMAC signing
├── app/channels/inertia_cable/
│   └── stream_channel.rb        # ActionCable channel (verifies signed tokens)
├── frontend/                    # React npm package (@inertia-cable/react)
│   └── src/
│       ├── useInertiaCable.ts   # Main React hook
│       ├── InertiaCableProvider.tsx # Context provider for ActionCable consumer
│       ├── consumer.ts          # ActionCable consumer singleton
│       └── index.ts             # Public exports
├── spec/                        # Ruby unit specs (RSpec)
├── integration_test/            # Full Rails app for integration testing
├── lib/generators/              # Rails install generator
├── inertia_cable.gemspec        # Gem specification
├── Gemfile                      # Dev dependencies
└── Rakefile                     # Default task: rspec
```

## Commands

### Ruby (gem)

```bash
bundle install                    # Install dependencies
bundle exec rspec                 # Run all Ruby specs
bundle exec rspec spec/broadcastable_spec.rb  # Run a single spec file
bundle exec rake                  # Same as bundle exec rspec (default task)
```

### Frontend (React package)

```bash
cd frontend
npm ci                            # Install dependencies (clean)
npm run build                     # Build with tsup
npm run typecheck                 # TypeScript type checking (tsc --noEmit)
npm test                          # Run tests (vitest run)
npm run test:watch                # Run tests in watch mode
npm run dev                       # Build in watch mode
```

### CI Pipeline

CI runs on every PR and push to main (`.github/workflows/ci.yml`):
- **ruby-specs**: Ruby 3.3, `bundle exec rspec`
- **frontend**: Node 22, `npm ci` + `npm run typecheck` + `npm run build`

## Architecture & Key Patterns

### Ruby Side

- **Broadcastable** (`lib/inertia_cable/broadcastable.rb`): ActiveSupport::Concern mixed into ActiveRecord::Base via the engine. Provides `broadcasts_to`, `broadcasts`, `broadcast_refresh_to`, `broadcast_message_to` and their `_later` variants.
- **Streams::StreamName**: Resolves streamables (models, strings, arrays) into canonical stream names and signs them with HMAC-SHA256 via `ActiveSupport::MessageVerifier`.
- **StreamChannel**: ActionCable channel that verifies signed tokens on subscription — rejects invalid/tampered tokens.
- **BroadcastJob**: ActiveJob wrapper for `_later` (async) broadcast variants.
- **Debounce**: Uses `Rails.cache` to coalesce rapid broadcasts within a configurable delay window.
- **Suppressor**: Thread-local flag to temporarily suppress all broadcasts (e.g., during seeds/migrations).
- **TestHelper**: Minitest-compatible assertions (`assert_broadcasts_on`, `assert_no_broadcasts_on`, `capture_broadcasts_on`) that work with both RSpec and Minitest.

### Frontend Side

- **useInertiaCable**: React hook that subscribes to a signed ActionCable stream, handles client-side debouncing, reconnection, and triggers `router.reload()` on refresh signals or calls `onMessage` for direct messages.
- **InertiaCableProvider**: React context provider wrapping the ActionCable consumer.
- Built with **tsup**, targeting both CJS and ESM outputs with type declarations.

### Payload Types

```json
// Refresh signal (triggers router.reload)
{ "type": "refresh", "model": "Message", "id": 42, "action": "create", "timestamp": "...", "extra": {} }

// Direct message (passed to onMessage callback)
{ "type": "message", "data": { ... } }
```

## Testing Conventions

### Ruby Specs

- Framework: RSpec with `.rspec` config (`--format documentation --color`)
- Database: In-memory SQLite — models and schemas defined inline in spec files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cole-robertson/inertia-cable](https://github.com/cole-robertson/inertia-cable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
