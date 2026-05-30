---
trigger: always_on
description: This is a **generator repository** that creates n8n community nodes from Apify Actors. It has two main components:
---

# AI Agent Context: n8n-nodes-apify-template

## Overview

This is a **generator repository** that creates n8n community nodes from Apify Actors. It has two main components:

1. **Template Files** (`nodes/ApifyActorTemplate/`) - Blueprint for generated nodes
2. **Generator Scripts** (`scripts/`) - Code that creates new nodes from the template

**Key Concept**: This is NOT a static codebase. When you run `npm run create-actor-app`, the scripts copy the template, fetch an Actor's schema from Apify, and generate a new node with proper naming and properties.

---

## Quick Commands

```bash
npm run create-actor-app  # Generate a new node from an Actor ID
npm run build             # Compile TypeScript + copy icons
npm run dev               # Run n8n locally with hot reload
npm test                  # Run tests (none exist yet)
npm run lint              # Check code quality
```

---

## Part 1: Template Files (What Gets Copied)

Location: `nodes/ApifyActorTemplate/`

These files serve as the blueprint for all generated nodes:

### Node Definition
- **`ApifyActorTemplate.node.ts`** - Main node class implementing `INodeType`
  - Defines node metadata (name, icon, credentials)
  - Contains `execute()` method that runs the Actor
  - Uses placeholders like `$$ACTOR_ID`, `$$CLASS_NAME` that get replaced

### Properties
- **`ApifyActorTemplate.properties.ts`** - UI field definitions
  - Gets **completely regenerated** during setup (not copied as-is)
  - Defines what fields users see in n8n UI

### Helpers
- **`helpers/executeActor.ts`** - Actor execution logic
  - `getDefaultBuild()` - Fetch Actor metadata
  - `getDefaultInputsFromBuild()` - Extract default values
  - `runActor()` - Execute Actor and return results

- **`helpers/genericFunctions.ts`** - API utilities
  - `apiRequest()` - Make authenticated calls to Apify API
  - `pollRunStatus()` - Poll every 1s until Actor completes
  - `getResults()` - Fetch dataset items
  - `isUsedAsAiTool()` - Detect if used by AI agents

- **`helpers/hooks.ts`** - n8n lifecycle hooks

### Metadata
- **`ApifyActorTemplate.node.json`** - Node categories and aliases
- **`properties.json`** - Cached generated properties

### Icons
- **`apify.svg`** / **`apifyDark.svg`** - Node icons (light/dark themes)

---

## Part 2: Generator Scripts (What Creates New Nodes)

Location: `scripts/`

### Main Flow
**`setupProject.ts`** - Orchestrates the entire generation process:
1. Prompts user for Actor ID
2. Calls `setConfig()` to create placeholder values
3. Calls `generateActorResources()` to convert schema
4. Calls `refactorProject()` to rename files/folders

### Key Scripts

**`actorConfig.ts`** - Generates placeholder values:
- `$$ACTOR_ID` → `apify/instagram-scraper`
- `$$CLASS_NAME` → `ApifyInstagramScraper`
- `$$DISPLAY_NAME` → `Apify Instagram Scraper`
- `$$PACKAGE_NAME` → `n8n-nodes-apify-instagram-scraper`
- `$$X_PLATFORM_APP_HEADER_ID` → `instagram-scraper-app`

**`actorSchemaConverter.ts`** - Converts Apify schema → n8n properties:

| Apify Type | Apify Editor | n8n Type | Notes |
|------------|-------------|----------|-------|
| string | (default) | string | Text input |
| string | textarea | string | Multi-line with `rows: 5` |
| string | select / enum | options | Dropdown |
| string | datepicker | dateTime | Date picker |
| integer | - | number | With min/max |
| boolean | - | boolean | Toggle |
| array | - | json or fixedCollection | Depends on items |
| object | - | json | JSON editor |

**`refactorProject.ts`** - Renames files and updates imports:
- Renames `ApifyActorTemplate/` → `ApifyActorName/`
- Updates all class names and imports
- Updates `package.json` name

**`createActorApp.ts`** - Fetches Actor metadata from Apify API

**`cli.ts`** - Entry point that calls `setupProject()`

---

## Generation Flow

```
User runs: npm run create-actor-app
      ↓
Prompt for Actor ID (e.g., "apify/instagram-scraper")
      ↓
Fetch Actor metadata via ApifyClient
      ↓
setConfig() → Generate placeholder values
      ↓
generateActorResources() → Convert Apify schema to n8n properties
      ↓
refactorProject() → Rename ApifyActorTemplate → ApifyInstagramScraper
      ↓
npm run build → Compile TypeScript
      ↓
Generated node ready in dist/
```

---

## Key Architecture Concepts

### n8n Node Structure
All n8n nodes must implement `INodeType`:
```typescript
export class ApifyActorName implements INodeType {
  description: INodeTypeDescription = { ... };
  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    // Run Actor and return results
  }
}
```

### Apify Actor Execution Flow
1. `getDefaultBuild()` → GET `/v2/acts/{actorId}/builds/default`
2. `getDefaultInputsFromBuild()` → Extract prefill values
3. `runActorApi()` → POST `/v2/acts/{actorId}/runs` (non-blocking)
4. `pollRunStatus()` → GET `/v2/actor-runs/{runId}` every 1s
5. `getResults()` → GET `/v2/datasets/{datasetId}/items`
6. Return formatted results to n8n

### Authentication
Two methods supported:
- **API Key** (`credentials/ApifyApi.credentials.ts`) - Bearer token
- **OAuth2** (`credentials/ApifyOAuth2Api.credentials.ts`) - PKCE flow

### AI Tool Support
- Nodes marked with `usableAsTool: true`
- `isUsedAsAiTool()` detects AI context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apify/n8n-nodes-apify-template](https://github.com/apify/n8n-nodes-apify-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
