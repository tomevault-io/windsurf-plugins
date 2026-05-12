---
trigger: always_on
description: Rails 8.1 + Inertia.js + React 19 + Vite 7 + Tailwind v4 + shadcn/ui, Ruby 4.0.1, TypeScript 5.9, SQLite3.
---

# DashBot

Rails 8.1 + Inertia.js + React 19 + Vite 7 + Tailwind v4 + shadcn/ui, Ruby 4.0.1, TypeScript 5.9, SQLite3.

## Commands

| Command | Purpose |
|---------|---------|
| `bin/dev` | Start Rails + Vite dev servers |
| `bin/rails test` | Rails Minitest suite |
| `bin/rails test:system` | System tests (Selenium + headless Chrome) |
| `npm test` | Vitest frontend suite |
| `npm run test:watch` | Vitest watch mode |
| `npm run check` | TypeScript type-check (`tsconfig.app.json` + `tsconfig.node.json`) |
| `bin/rubocop` | Ruby linting (rubocop-rails-omakase) |
| `bin/brakeman` | Security scan |
| `bin/bundler-audit` | Gem vulnerability audit |
| `bin/ci` | Full CI: setup, rubocop, bundler-audit, brakeman, rails test, seed |
| `npx shadcn@latest add <name>` | Add shadcn/ui component |

## Architecture

### Inertia bridge

Controllers render `render inertia: "folder/component", props: { ... }`. The entrypoint (`app/frontend/entrypoints/inertia.tsx`) resolves pages via `import.meta.glob('../pages/**/*.tsx', { eager: true })`, so `"home/index"` maps to `app/frontend/pages/home/index.tsx`.

### Directory layout

```
app/
  controllers/
    concerns/authentication.rb   # Session-based auth, included by ApplicationController
    auth_controller.rb            # Login (skips require_authentication)
    home_controller.rb
  models/
    user.rb, profile.rb, chat_session.rb, message.rb, qr_token.rb, setting.rb
  frontend/
    entrypoints/inertia.tsx       # Inertia app bootstrap (React 19 createRoot, StrictMode)
    pages/                        # Inertia page components (default exports)
      auth/login.tsx, auth/qr_login.tsx
      home/index.tsx
    components/ui/                # shadcn/ui components
    lib/utils.ts                  # cn() helper
    styles/app.css                # Tailwind v4 theme + shadcn CSS variables
    types/index.ts                # Shared TypeScript types
    test/                         # Vitest tests (mirrors source structure)
test/
  test_helper.rb
  application_system_test_case.rb # Base class for system tests (removes Vite stubs)
  fixtures/                       # Minitest fixtures
  models/, integration/           # Rails tests
  system/                         # System tests (Selenium + headless Chrome)
config/
  ci.rb                           # CI step definitions
```

### Key wiring

- **Path aliases**: `@/` and `~/` both resolve to `app/frontend/` (tsconfig.app.json + vitest.config.ts)
- **Auth**: Session-based via `Authentication` concern — stores `session[:user_id]` and `session[:profile_id]`, provides `current_user` and `current_profile` helpers
- **Inertia types**: `app/frontend/types/index.ts`
- **Background jobs**: solid_queue; **Caching**: solid_cache

## Status Page

Real-time OpenClaw agent monitoring at `/status`. Data flows entirely over WebSocket — no polling.

### Architecture

```
Browser (StatusChannel)  ←→  Rails Action Cable  ←→  Plugin (ChatChannel)
         ↑                                                     ↑
    status_updates                                     plugin_commands
```

### Flow

1. **Plugin connects** — `dashbot-openclaw` plugin connects via WebSocket, subscribes to `ChatChannel`. All ChatChannel subscriptions also stream from `plugin_commands`.
2. **Viewer opens `/status`** — frontend subscribes to `StatusChannel`. First viewer triggers `status_requested` broadcast to `plugin_commands`.
3. **Plugin receives request** — starts `StatusReporter` which reads OpenClaw state files (sessions, cron jobs, memory DB) and sends `send_status` action every 15s.
4. **Rails relays data** — `ChatChannel#send_status` calls `StatusChannel.broadcast_status` (WebSocket push to all viewers) and writes to `Rails.cache` (for initial page loads).
5. **Viewer leaves** — last viewer unsubscribing triggers `status_stopped` → plugin stops reporting.

### Key files

| File | Role |
|------|------|
| `app/channels/status_channel.rb` | Viewer count tracking, request/stop lifecycle |
| `app/channels/chat_channel.rb` | Plugin connection, relays `send_status` to StatusChannel + cache |
| `app/controllers/status_controller.rb` | Initial page load (from cache), poll endpoint, keepalive |
| `app/frontend/pages/status/index.tsx` | React status dashboard, StatusChannel subscription |
| `app/controllers/api/status_controller.rb` | API endpoint for status data from OpenClaw plugin |
| `app/controllers/api/cards_controller.rb` | API endpoint for agentic card push + response |
| `app/controllers/api/messages_controller.rb` | API endpoint for assistant messages from agent |

## Code Conventions

### Rails

- `frozen_string_literal: true` on all Ruby files
- `has_secure_password` with `alias_attribute :password_digest, :encrypted_password`
- Controllers include `Authentication` concern; skip with `skip_before_action :require_authentication`
- Inertia rendering: `render inertia: "folder/component", props: { ... }`
- Rubocop: `rubocop-rails-omakase` style, no custom overrides

### React / TypeScript

- Pages are default exports with `interface Props`
- Import via `@/` alias (e.g., `@/components/ui/button`)
- `router` from `@inertiajs/react` for navigation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wembledev/dashbot](https://github.com/wembledev/dashbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
