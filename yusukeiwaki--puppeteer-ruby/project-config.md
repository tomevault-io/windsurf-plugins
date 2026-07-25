---
trigger: always_on
description: > **IMPORTANT: Spec Migration Tracking**
---

# Puppeteer-Ruby Development Guide

> **IMPORTANT: Spec Migration Tracking**
>
> Test porting from Node.js Puppeteer to Ruby RSpec is tracked in **[CLAUDE/spec_migration_plans.md](./CLAUDE/spec_migration_plans.md)**.
>
> - Before porting tests: Check the migration plan for priorities and status
> - After porting tests: Update the migration plan to reflect progress
> - This file must be kept up-to-date with all spec migration work

This document provides essential guidance for AI agents working on the puppeteer-ruby codebase.

## Project Overview

puppeteer-ruby is a Ruby port of [Puppeteer](https://pptr.dev/), the Node.js browser automation library. It uses the Chrome DevTools Protocol (CDP) to automate Chrome/Chromium browsers.

### Core Principles

1. **CDP Protocol Focus**: All browser automation is done via CDP
2. **Chrome Specialization**: Focused on Chrome/Chromium automation
3. **API Compatibility**: Follow Puppeteer's API design closely, but use Ruby idioms

## Quick Reference

### Running Tests

```bash
# Run all tests
bundle exec rspec

# Run specific test file
bundle exec rspec spec/integration/page_spec.rb

# Run in debug mode (non-headless)
DEBUG=1 bundle exec rspec spec/integration/page_spec.rb
```

> **Note for Codex CLI**: When executing RSpec from Codex CLI, always use `rbenv exec`:
> ```bash
> rbenv exec bundle exec rspec spec/integration/click_spec.rb
> ```

### Key Environment Variables

| Variable | Description |
|----------|-------------|
| `PUPPETEER_EXECUTABLE_PATH_RSPEC` | Custom browser executable path |
| `PUPPETEER_CHANNEL_RSPEC` | Chrome channel (e.g., `chrome`, `chrome-beta`) |
| `DEBUG` | Set to `1` for debug output |
| `PUPPETEER_NO_SANDBOX_RSPEC` | Add `--no-sandbox` flag (for containers) |

### Code Quality

```bash
# Run RuboCop
bundle exec rubocop

# Auto-fix RuboCop issues
bundle exec rubocop -a

# Run Steep type check
bundle exec steep check
```

## Architecture

The codebase follows a straightforward architecture:

```
lib/puppeteer/
├── puppeteer.rb          # Main entry point (Puppeteer.launch, Puppeteer.connect)
├── browser.rb            # Browser instance management
├── browser_context.rb    # Incognito/default context
├── page.rb               # Page API (main user-facing class)
├── frame.rb              # Frame handling
├── element_handle.rb     # DOM element operations
├── js_handle.rb          # JavaScript object handles
├── connection.rb         # WebSocket connection to browser
├── cdp_session.rb        # CDP session management
├── keyboard.rb           # Keyboard input simulation
├── mouse.rb              # Mouse input simulation
└── ...
```

### Key Components

- **Connection**: Manages WebSocket connection to the browser's DevTools
- **CDPSession**: Sends CDP commands and receives events
- **FrameManager**: Tracks frames and their execution contexts
- **NetworkManager**: Handles request interception and network events
- **LifecycleWatcher**: Waits for navigation events (load, DOMContentLoaded, etc.)

## Code Standards

### Ruby Version & Style

- Minimum Ruby version: 3.2
- Follow RuboCop rules defined in `.rubocop.yml`
- Use explicit keyword arguments for public APIs

### API Naming Conventions

JavaScript Puppeteer methods use camelCase, Ruby methods use snake_case:

| Puppeteer (JS) | puppeteer-ruby |
|----------------|----------------|
| `page.waitForSelector()` | `page.wait_for_selector` |
| `page.setContent()` | `page.content=` or `page.set_content` |
| `element.boundingBox()` | `element.bounding_box` |
| `browser.newPage()` | `browser.new_page` |

### Error Classes

All custom errors inherit from `Puppeteer::Error`:

```ruby
module Puppeteer
  class Error < StandardError; end
  class TimeoutError < Error; end
  class FrameNotFoundError < Error; end
  # etc.
end
```

## Testing Strategy

### Test Types

- **Unit tests**: `spec/puppeteer/` - Test individual classes without browser
- **Integration tests**: `spec/integration/` - Test with real browser

### Integration Test Setup

Integration tests use RSpec metadata:

```ruby
RSpec.describe 'Page', type: :puppeteer do
  it 'navigates to a page' do
    page.goto('https://example.com')
    expect(page.title).to eq('Example Domain')
  end
end
```

The `type: :puppeteer` metadata automatically:
- Launches Chrome before each test
- Provides `page` helper method
- Closes browser after test

## Porting from Puppeteer

When implementing new features, reference the TypeScript Puppeteer source:

1. Find the corresponding TypeScript file in [puppeteer/puppeteer](https://github.com/puppeteer/puppeteer)
2. Understand the CDP calls being made
3. Implement in Ruby following existing patterns
4. Port the relevant tests
5. Update `docs/api_coverage.md`

### CDP Command Pattern

```ruby
# TypeScript Puppeteer
await this._client.send('Page.navigate', { url });

# Ruby equivalent
@client.send_message('Page.navigate', url: url)
```

## Concurrency Model

### Current State (socketry/async)

puppeteer-ruby uses Fiber-based concurrency with `socketry/async` (version 2.35.1+):

- `Async::Promise` - For async operations that complete later
- `Async` blocks - For running operations in Fiber context
- `Puppeteer::AsyncUtils.await_promise_all` - For waiting on multiple promises

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YusukeIwaki/puppeteer-ruby](https://github.com/YusukeIwaki/puppeteer-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
