---
trigger: always_on
description: > **Last Updated**: January 6, 2026
---

# Story UI - AI Assistant Project Guide

> **Last Updated**: January 6, 2026
> **Current Version**: 4.6.3
> **Production URL**: https://app-production-16de.up.railway.app (Vue/Vuetify example)
> **Repository**: https://github.com/southleft/story-ui

This document provides comprehensive context for AI assistants working on the Story UI codebase. It captures what the project is, how it works, architecture decisions, and development workflows to minimize token consumption during codebase analysis.

---

## What is Story UI?

**Story UI is an AI-powered Storybook story generator that works with ANY component library.** Users describe components in natural language, and the AI generates working Storybook stories using their design system's actual components.

### Core Value Proposition

- **Design-System Agnostic**: Works with React (Mantine, Chakra, MUI), Vue (Vuetify), Angular (Material), Svelte (Flowbite), Web Components (Shoelace)
- **Natural Language Interface**: "Create a card with a header, image, and action buttons"
- **Live Preview**: Generated stories appear instantly in Storybook
- **Multi-Provider LLM**: Supports Claude, OpenAI, and Gemini
- **Self-Healing Code Generation**: Validates generated code and auto-corrects errors via LLM retry loop

---

## Quick Reference

### Important Files

| Purpose | Location |
|---------|----------|
| MCP Server (Express) | `mcp-server/index.ts` |
| STDIO MCP Server | `mcp-server/mcp-stdio-server.ts` |
| Story Generation | `mcp-server/routes/generateStory.ts` |
| Streaming Generation | `mcp-server/routes/generateStoryStream.ts` |
| Self-Healing Loop | `story-generator/selfHealingLoop.ts` |
| Component Discovery | `story-generator/componentDiscovery.ts` |
| LLM Providers | `story-generator/llm-providers/` |
| Framework Adapters | `story-generator/framework-adapters/` |
| Storybook Panel | `templates/StoryUI/StoryUIPanel.tsx` |
| MDX Wrapper | `templates/StoryUI/StoryUIPanel.mdx` |
| CLI Entry | `cli/index.ts` |
| CLI Setup | `cli/setup.ts` |

### Quick Commands

```bash
# Build the package
npm run build

# Start MCP server locally
npm run story-ui

# Watch mode for development
npm run dev

# Run in test environment
cd /path/to/test-storybooks/react-mantine
PORT=4101 node /path/to/story-ui/dist/mcp-server/index.js
```

---

## Test Storybook Environments

Development and testing uses five framework-specific Storybook instances (create these in a sibling directory to story-ui):

| Directory | Framework | Design System | Storybook Port | MCP Port |
|-----------|-----------|---------------|----------------|----------|
| `react-mantine` | React 19 | Mantine 8.x | 6101 | 4101 |
| `angular-material` | Angular 21 | Material 21 | (ng run) | 4102 |
| `vue-vuetify` | Vue 3 | Vuetify 3.x | 6103 | 4103 |
| `svelte-flowbite` | Svelte 5 | Flowbite + Tailwind | 6104 | 4104 |
| `web-components-shoelace` | Lit 3 | Shoelace 2.x | 6105 | 4105 |

### Starting a Test Environment

```bash
# Example: React Mantine
cd ../test-storybooks/react-mantine

# Terminal 1: Start MCP server
PORT=4101 node ../story-ui/dist/mcp-server/index.js

# Terminal 2: Start Storybook
npm run storybook -- --port 6101
```

### Port Convention

- **Storybook**: 6100 series (6101, 6102, 6103, 6104, 6105)
- **MCP Server**: 4100 series (4101, 4102, 4103, 4104, 4105)

---

## MCP Server Architecture

### Two Operation Modes

**1. HTTP Server** (Primary for web/local development)
```
npm run story-ui  →  Express server on PORT (default: 4001)
                  →  Serves API endpoints
                  →  Optional Storybook proxy mode
```

**2. STDIO Server** (For Claude Desktop integration)
```
npm run mcp  →  MCP Server using stdio transport
             →  Makes HTTP calls to local HTTP server
             →  Requires HTTP server running on port 4001
```

### Server Startup Flow

```
1. Load .env configuration
2. Create Express app
3. Apply CORS middleware
4. Register API routes:
   - /mcp/generate-story (POST) - Story generation
   - /mcp/generate-story-stream (POST) - Streaming generation
   - /mcp/components (GET) - Component discovery
   - /mcp/providers (GET) - Available LLM providers
   - /story-ui/* - Aliased routes (proxy to /mcp/*)
   - /mcp-remote/* - Claude Desktop MCP endpoint
5. Load user configuration (story-ui.config.js)
6. Optional: Configure Storybook proxy (if STORYBOOK_PROXY_ENABLED=true)
7. Start listening on PORT
```

### API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/mcp/components` | GET | List discovered components |
| `/mcp/generate-story` | POST | Generate story from prompt |
| `/mcp/generate-story-stream` | POST | Streaming story generation |
| `/mcp/providers` | GET | List available LLM providers |
| `/mcp/providers/models` | GET | List models per provider |
| `/story-ui/stories` | GET/POST | Story file management |
| `/mcp-remote/*` | POST | Claude Desktop MCP endpoint |

### Port Configuration Priority

The `StoryUIPanel.tsx` determines MCP server URL in this order:
1. `VITE_STORY_UI_EDGE_URL` - Cloud deployment
2. `window.__STORY_UI_EDGE_URL__` - Runtime override
3. Railway hostname detection - Same origin
4. `VITE_STORY_UI_PORT` - From .env file
5. `window.__STORY_UI_PORT__` - Legacy override
6. `window.STORY_UI_MCP_PORT` - MDX wrapper override

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [southleft/story-ui](https://github.com/southleft/story-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
