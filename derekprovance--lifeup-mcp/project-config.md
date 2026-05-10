---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LifeUp MCP Server** is a Model Context Protocol (MCP) server that enables Claude to interact with the LifeUp task management app running on a local Android device. The server acts as a bridge between Claude and LifeUp's HTTP API, exposing 20 tools for task creation, achievement management, querying, user profile information, shop browsing, and data mutation operations. In SAFE_MODE, 14 tools are available (11 read-only + 3 create operations).

## Architecture

### High-Level Design

```
Claude (via MCP)
    ↓
MCP Server (Node.js/TypeScript)
    ├── Request Handlers (server.ts)
    ├── Tool Implementations (tools/)
    ├── API Client (client/lifeup-client.ts)
    ├── Configuration (config/)
    ├── Error Handling (error/)
    └── Validation (via Zod)
    ↓
HTTP Client (axios)
    ↓
LifeUp Cloud API (Android Device)
```

### Key Components

**server.ts** - MCP server initialization and tool registration
- Registers two request handlers: `tools/list` and `tools/call`
- Dispatches tool calls to appropriate tool implementations
- Uses Zod schemas for request validation

**client/lifeup-client.ts** - HTTP wrapper around LifeUp's REST API
- Singleton instance (`lifeupClient`)
- Handles all HTTP requests to LifeUp server
- Converts axios errors to `LifeUpError` with user-friendly messages
- Health check capability for connectivity verification

**tools/task-tools.ts** - Task management operations
- 6 exported static methods: `createTask`, `listAllTasks`, `searchTasks`, `getTaskHistory`, `getTaskCategories`, `deleteTask`
- All return formatted markdown strings for Claude
- Calls `lifeupClient` methods and wraps results in error handling

**tools/achievement-tools.ts** - Achievement querying, matching, and management
- `listAchievements` - Lists all achievements by fetching from all categories (N+1 requests where N=category count), with fallback to categories if unavailable
  - Shows unlock conditions when available (requires API support)
  - Displays 21 condition types: task completions, coins, skills, items, usage metrics, etc.
  - Format: "Unlock by: [condition 1] AND [condition 2]"
  - Falls back gracefully if conditions unavailable
  - Shows detailed progress for locked achievements (e.g., "50% progress: 5/10")
- `listAchievementCategories` - Lists all achievement categories with IDs and descriptions
- `matchTaskToAchievements` - Keyword-based matching algorithm to suggest relevant achievements for a task
- `createAchievement` - Create new custom achievements with optional unlock conditions and rewards
- `updateAchievement` - Modify existing achievement properties (name, description, rewards, unlock status). ⚠️ IMPORTANT: The LifeUp API does not support updating conditions_json. To change unlock conditions, you must delete the achievement and create a new one.
- `deleteAchievement` - Delete achievement definitions permanently

**tools/user-info-tools.ts** - User profile and character information
- `listSkills` - Lists all skills with levels, experience, and progress to next level
- `getUserInfo` - Displays player name, character level, version, and total experience
- `getCoinBalance` - Shows current coin balance and currency information
- Helps users understand their character progression and economy

**tools/shop-tools.ts** - Shop browsing and item search
- `listShopItems` - Lists all shop items with prices, stock availability, and owned quantities
- `getShopCategories` - Lists all shop item categories for organization
- `searchShopItems` - Filters items by name, category, and price range
- Enables users to browse and search the shop inventory

**tools/mutation-tools.ts** - Safe data mutation operations
- `editTask` - Edit existing task properties (name, rewards, category, appearance, task type, count settings)
- `addShopItem` - Create new shop items with price, stock, effects, and purchase limits
- `editShopItem` - Modify shop items with absolute/relative adjustments
- `applyPenalty` - Apply penalties (coins, exp, or items) with custom reasons
- `editSkill` - Create, update, or delete skills; adjust skill experience points
- All operations are "safe" - they don't auto-grant rewards or auto-complete tasks

**config/config.ts** - Configuration singleton
- Loads environment variables (LIFEUP_HOST, LIFEUP_PORT, LIFEUP_API_TOKEN, DEBUG, SAFE_MODE)
- Provides `configManager` singleton
- Debug logging utility for troubleshooting
- Safe mode support for disabling mutation tools

**config/validation.ts** - Zod schemas for input validation
- `CreateTaskSchema`, `SearchTasksSchema`, `TaskHistorySchema`, `AchievementMatchSchema`, `SearchShopItemsSchema`
- All tool inputs validated before execution

**error/error-handler.ts** - Error handling utilities
- `LifeUpError` class with technical and user-facing messages
- `ErrorHandler.handleNetworkError()` - Maps axios errors to LifeUpError
- `ErrorHandler.handleApiError()` - Handles HTTP response errors
- Graceful fallback for API features that may not be available

**client/types.ts** - TypeScript interfaces for all API response types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derekprovance/lifeup-mcp](https://github.com/derekprovance/lifeup-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
