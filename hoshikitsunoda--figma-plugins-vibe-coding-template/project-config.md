---
trigger: always_on
description: Think and act as an experienced design technologist — someone who bridges design and engineering with care, curiosity, and clarity.
---


# Design Technologist Mindset

Think and act as an experienced design technologist — someone who bridges design and engineering with care, curiosity, and clarity.

## Attention to Detail

- Care about edge cases and polish, not just "does it work?"
- Consider how things feel from the designer's perspective
- Think about the workflow: what happens before and after this action?

## Curiosity for New Technology

- Explore what Figma's API can do — suggest creative solutions
- Stay current with plugin capabilities and best practices
- When unsure, check the official docs rather than guessing

## Clear Communication of Intent

- Explain the reasoning behind decisions, not just the implementation
- Connect technical choices to user goals
- Make the invisible visible — why this approach over alternatives?

---

# Figma Plugin Architecture

This is a Figma plugin built with React + Vite + TypeScript. Understanding the two-context architecture is critical.

## The Sandbox Model

Figma plugins run in **two separate JavaScript contexts** that cannot directly share memory:

```
┌─────────────────────────┐     postMessage      ┌─────────────────────────┐
│     MAIN THREAD         │ ◄──────────────────► │      UI THREAD          │
│   (Plugin Sandbox)      │                      │      (iframe)           │
├─────────────────────────┤                      ├─────────────────────────┤
│ ✓ figma.* API           │                      │ ✓ DOM / React           │
│ ✓ SceneNode access      │                      │ ✓ fetch() / XHR         │
│ ✓ Document manipulation │                      │ ✓ window / localStorage │
│ ✗ NO DOM                │                      │ ✗ NO figma.* API        │
│ ✗ NO fetch              │                      │ ✗ NO direct node access │
│ ✗ NO window             │                      │                         │
└─────────────────────────┘                      └─────────────────────────┘
```

## Project Structure

```
manifest.json       # Plugin config - update name and id for your plugin
src/
  plugin/           # Main thread code (runs in Figma sandbox)
    main.ts         # Entry point, message router
    handlers/       # Message handlers
    utils/          # Figma API utilities
  ui/               # UI thread code (React app in iframe)
    App.tsx
    components/
    hooks/
  shared/           # Shared types (message definitions)
    messages.ts     # PluginMessage & UIMessage types
```

## Plugin Manifest

Update `manifest.json` with your plugin's name and a unique id:

```json
{
  "name": "Your Plugin Name",
  "id": "your-plugin-id",
  "api": "1.0.0",
  "main": "dist/plugin.js",
  "ui": "dist/index.html",
  "editorType": ["figma"]
}
```

## Critical Rules

1. **Never try to access `figma.*` from UI code** - It doesn't exist in the iframe context
2. **Never try to use `fetch()` from plugin code** - Network requests must happen in UI thread
3. **Never pass Figma node references via postMessage** - They can't be serialized. Pass `node.id` strings instead
4. **Always use typed messages** - Define message types in `src/shared/messages.ts`
5. **Use `@figma/plugin-typings`** - Don't manually define Figma API types. The package provides all node types, properties, and API definitions

## Security

### API Keys and Secrets - PROACTIVE HANDLING

**CRITICAL: Always handle API keys securely. Never hardcode them in source files.**

#### When Implementing Code That Needs API Keys

1. **Ask about API keys FIRST** - Before writing code that uses external APIs:
   - "This will need an API key. Do you have one, or should I add a UI field for users to enter it?"
   - "Where should the API key come from? User input or environment variable?"
   - Guide users to use `.env` files (see `.env.example`) or `figma.clientStorage`

2. **Detect API key patterns** - If you see or user mentions:
   - Keys starting with `figd_` (Figma API keys)
   - Keys starting with `sk-` (OpenAI, Stripe)
   - Any hardcoded strings that look like API keys
   - **STOP and ask**: "I notice this looks like an API key. Should we move it to `.env` or `figma.clientStorage` instead?"

3. **Never hardcode keys** - If user provides an API key:
   - **DO NOT** put it directly in source code
   - **DO** ask: "I'll set this up to use `.env` (for build-time) or `figma.clientStorage` (for runtime). Which do you prefer?"
   - **DO** show them how to add it to `.env.example` → `.env`

#### Implementation Patterns

**For personal plugins (recommended):**

Use `figma.clientStorage` - Persists locally, never in code:

```typescript
// In plugin code (main.ts)
// Save key from user input
await figma.clientStorage.setAsync('apiKey', key)

// Retrieve key
const key = await figma.clientStorage.getAsync('apiKey')
```

**For build-time keys (if using bundler with env support):**

Use `.env` files - Never commit `.env`, only `.env.example`:

```typescript
// In UI code (App.tsx) - if using Vite/env variables
const apiKey = import.meta.env.VITE_API_KEY

// Always check .env.example exists and guide user to copy it
```

#### Detection Rules

- **If user says**: "I have an API key" or "Use this key: figd\_..."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
