---
trigger: always_on
description: This is **Cognigy Webchat v3** - a drop-in chat widget for Cognigy.AI endpoints. The widget is built with React + Redux + Emotion (CSS-in-JS) and compiled to standalone UMD/ESM bundles that embed into any website via `<script>` tags.
---

# Cognigy Webchat v3 - AI Agent Instructions

## Project Overview

This is **Cognigy Webchat v3** - a drop-in chat widget for Cognigy.AI endpoints. The widget is built with React + Redux + Emotion (CSS-in-JS) and compiled to standalone UMD/ESM bundles that embed into any website via `<script>` tags.

**Critical Architectural Constraint**: Only ONE webchat instance per page. Not designed for multiple sessions (parallel or sequential).

## Architecture

### Three-Layer Structure

1. **`src/webchat`** - Core React component with Redux state management + socket connection logic
2. **`src/webchat-ui`** - Presentational UI components using `@cognigy/chat-components`
3. **`src/webchat-embed`** - Entry point that mounts the widget to DOM and exposes `window.initWebchat()`

**Entry point**: `src/webchat-embed/index.tsx` loads plugins, prepares message/input plugins, and initializes the Webchat component.

### State Management

- **Mixed Redux approach**: Older reducers use hand-written reducers (see `src/webchat/store/config/config-reducer.ts`), newer ones use Redux Toolkit's `createSlice()` (see `src/webchat/store/typing/slice.ts`, `xapp-overlay/slice.ts`)
- **Root reducer**: `src/webchat/store/reducer.ts` combines all reducers with `combineReducers()` and handles special actions like `RESET_STATE` (restore persisted history) and `SET_PREV_STATE`
- **Key stores**: messages, config, ui, connection, rating, input, prevConversations, unseenMessages, inputCollation, autoInject, xAppOverlay, queueUpdates, userTyping

### Plugin System

Webchat supports **Message Plugins** (render custom message types) and **Input Plugins** (custom input UI):

- **Registration**: Plugins call `registerMessagePlugin()` or `registerInputPlugin()` from `src/plugins/helper.tsx` - these push to global `window.cognigyWebchatMessagePlugins` / `window.cognigyWebchatInputPlugins` arrays
- **Loading**: Import plugins in `src/webchat-embed/index.tsx` BEFORE mounting (e.g., `import "../plugins/rating"`)
- **Message matching**: Plugins define a `match` function `(message, config) => boolean` - typically checks `message.data._plugin.type`
- **Dependency injection pattern**: Plugins receive `{ React, styled }` at runtime to share host dependencies (avoids bundling React twice)

**Built-in plugins**: `src/plugins/get-started-button-input/`, `src/plugins/rating/`

External plugins: See [WebchatPlugins repo](https://github.com/Cognigy/WebchatPlugins) for examples. Plugins use `alias/react.js` pattern to consume host's React.

## Configuration System

**Adding new config options** (see `architecture-docs/quality-attributes.md`):

1. Add property to `IWebchatConfig` or `IWebchatSettings` in `src/common/interfaces/webchat-config.ts`
2. Set default value in `getInitialState()` in `src/webchat/store/config/config-reducer.ts`
3. Access via Redux `state.config.settings.*` throughout the app

Settings come from Endpoint Config API (merged with client-side `initWebchat()` options).

## Styling

- **Emotion CSS-in-JS**: All components use `styled()` from `@emotion/styled` - see examples in `src/webchat-ui/components/`
- **Theme**: Injected via Emotion's `ThemeProvider` (colors, spacing) - see `src/webchat-ui/style`
- **CSS classes**: Exposed for external customization - documented in `docs/css-customization.md` (e.g., `webchat-root`, `webchat-header-bar`, `webchat-message-row`)
- **Components library**: Uses `@cognigy/chat-components` for base UI (Typography, Button, ActionButtons, Message, etc.)

## Build System

**Webpack configurations**:

- `webpack.config.js` - Base config (dev mode)
- `webpack.production.js` - UMD bundle (`dist/webchat.js`)
- `webpack.es.js` - ESM bundle (`dist/webchat.esm.js`) with external React/React-DOM
- `webpack.dev.js` - Dev server config

**Build process**:

```bash
npm run build       # Builds both UMD + ESM, runs postbuild-secure-patch
npm run dev         # Webpack dev server on localhost:8080
```

**Post-build security patching**: `scripts/postbuild-secure-patch.mjs` applies idempotent text replacements to fix CodeQL findings (incomplete sanitization, regex issues) - runs automatically after build.

## Development Workflows

### Running Locally

```bash
npm install
npm run dev         # Starts dev server at http://localhost:8080
```

Opens test page with webchat embedded. **Hot reload enabled** - changes auto-refresh.

### Testing

```bash
npm run build                          # Must build first
npm run test                          # Runs Cypress tests (Chrome)
npm run test:cypress:firefox          # Firefox variant
npm run test:cypress:progressive-rendering  # Tests progressive rendering mode
```

**Cypress architecture** (see `cypress/support/commands.ts`):

- `cy.initMockWebchat()` - Mocks endpoint, initializes webchat, stores instance in `window.webchat`
- `cy.receiveMessage()` - Simulates receiving a message from socket (exposed internal function)
- Tests run against built bundles served by `http-server` on port 8787

### Type Checking & Linting

```bash
npm run tsc:check      # TypeScript type checking (no emit)
npm run lint           # ESLint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cognigy/Webchat](https://github.com/Cognigy/Webchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
