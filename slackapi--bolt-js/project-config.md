---
trigger: always_on
description: Instructions for AI coding agents working on this repository.
---

# AGENTS.md - bolt-js

Instructions for AI coding agents working on this repository.

## Project Overview

Slack Bolt for JavaScript -- a framework for building Slack apps, fast.

- **Foundation:** Built on top of `@slack/web-api`, `@slack/oauth`, `@slack/socket-mode`, and other `@slack/*` packages (see `package.json` for the full list and versions).
- **Language:** TypeScript-first, compiled to CommonJS.
- **Node version:** See `engines` in `package.json` for minimum Node.js and npm versions.
- **Repository**: <https://github.com/slackapi/bolt-js>
- **Documentation**: <https://docs.slack.dev/tools/bolt-js/>
- **npm**: <https://www.npmjs.com/package/@slack/bolt>
- **Current version**: defined in `package.json`
- **Examples**: Sample apps in `examples/` (Socket Mode, OAuth, Lambda, custom receivers, etc.)

## Environment Setup

```bash
# Install all dependencies
npm install
```

## Common Commands

Before considering any work complete, you MUST run `npm test` and confirm it passes.

```bash
npm test              # Full pipeline: build -> lint -> type tests -> unit test coverage
npm run build         # Clean build (rm dist/ + tsc compilation)
npm run lint          # Biome check (formatting + linting)
npm run lint:fix      # Biome auto-fix
npm run test:unit     # Unit tests only (mocha)
npm run test:coverage # Unit tests with coverage (c8)
npm run test:types    # Type definition tests (tsd)
npm run watch         # Watch mode for development (rebuilds on src/ changes)
```

## Architecture

### Event Flow

Incoming events flow through a middleware chain before reaching listeners:

1. **Receiver** ingests event from Slack (HTTP, Socket Mode, or Lambda)
2. Receiver calls `App.processEvent(ReceiverEvent)`
3. **Global middleware** chain executes (authorization, self-event ignoring, custom middleware)
4. App determines event type and matches relevant **listeners** based on constraints
5. **Listener-specific middleware** chains execute
6. **Listener handler** runs with full context

For FaaS environments (`processBeforeResponse: true`), acknowledgment happens after the handler executes.

### Core Classes

- **`App`** (`src/App.ts`) -- Central orchestrator. Registers listeners via methods (`app.event()`, `app.action()`, `app.command()`, etc.). Dispatches incoming events through middleware to matching listeners. Manages the Web API client pool, authorization, and error handling.
- **`Receiver`** interface (`src/types/receiver.ts`) -- Pluggable transport layer abstraction. Methods: `init(app)`, `start()`, `stop()`.
  - `HTTPReceiver` (`src/receivers/HTTPReceiver.ts`) -- Express v5 HTTP server, default receiver
  - `SocketModeReceiver` (`src/receivers/SocketModeReceiver.ts`) -- WebSocket-based, no public URL needed
  - `ExpressReceiver` (`src/receivers/ExpressReceiver.ts`) -- Integrates with an existing Express v5 app
  - `AwsLambdaReceiver` (`src/receivers/AwsLambdaReceiver.ts`) -- AWS Lambda handler
- **`Assistant`** (`src/Assistant.ts`) -- AI assistant thread handling middleware. Intercepts assistant thread events (`assistant_thread_started`, `assistant_thread_context_changed`, `message` in assistant threads) and dispatches them to registered sub-handlers. Provides utilities: `say`, `setStatus`, `setSuggestedPrompts`, `setTitle`, `getThreadContext`, `saveThreadContext`. Uses `AssistantThreadContextStore` (`src/AssistantThreadContextStore.ts`) for thread context persistence.
- **`CustomFunction`** (`src/CustomFunction.ts`) -- Workflow custom function handler. Provides `complete()` and `fail()` utilities for function execution lifecycle.
- **`WorkflowStep`** (`src/WorkflowStep.ts`) -- **Deprecated.** Use `CustomFunction` and `app.function()` instead.

### Middleware System

Middleware uses a chain-of-responsibility pattern. Each middleware receives args and calls `next()` to continue the chain.

**Type:** `Middleware<Args> = (args: Args & AllMiddlewareArgs) => Promise<void>`

**AllMiddlewareArgs** (always available):

- `context` -- Event metadata (botToken, userToken, botId, botUserId, teamId, enterpriseId, etc.)
- `logger` -- Logger instance
- `client` -- Web API client (pre-authorized)
- `next` -- Call to continue the middleware chain

**Built-in middleware** in `src/middleware/builtin.ts` includes constraint matchers (event type, command name, message pattern, action/shortcut/view constraints), type guards (`onlyActions`, `onlyCommands`, etc.), `ignoreSelf`, and `autoAcknowledge`.

### Listener Methods

| Method | Description | Must `ack()`? |
|--------|-------------|---------------|
| `app.event(type, fn)` | Events API events | No |
| `app.message([pattern,] fn)` | Message events (optional string/RegExp filter) | No |
| `app.action(constraints, fn)` | Block Kit interactive actions (buttons, selects, etc.) | Yes |
| `app.command(name, fn)` | Slash commands | Yes |
| `app.shortcut(constraints, fn)` | Global and message shortcuts | Yes |
| `app.view(constraints, fn)` | Modal view_submission / view_closed | Yes |
| `app.options(constraints, fn)` | External data source requests | Yes |
| `app.function(callbackId, fn)` | Custom workflow function executions | Auto-acknowledged |

### Listener Arguments

Listeners receive a single object with these properties (availability depends on event type):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slackapi/bolt-js](https://github.com/slackapi/bolt-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
