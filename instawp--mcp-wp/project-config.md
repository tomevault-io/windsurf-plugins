---
trigger: always_on
description: Handles ALL content types (posts, pages, custom post types) with a single set of tools:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WordPress MCP (Model Context Protocol) server that allows interaction with WordPress sites through natural language via MCP-compatible clients like Claude Desktop. The server exposes WordPress REST API functionality as MCP tools.

## Development Commands

### Build and Run
```bash
# Install dependencies
npm install

# Build TypeScript to JavaScript
npm run build

# Run in development mode with hot reload
npm run dev

# Run the built server
npm start

# Clean build artifacts
npm run clean
```

### Environment Setup

#### Single Site Configuration
Create a `.env` file in the project root with:
```env
WORDPRESS_API_URL=https://your-wordpress-site.com
WORDPRESS_USERNAME=wp_username
WORDPRESS_PASSWORD=wp_app_password
```

#### Multi-Site Configuration
For managing multiple WordPress sites:
```env
# Site 1 (Production)
WORDPRESS_1_URL=https://production-site.com
WORDPRESS_1_USERNAME=admin
WORDPRESS_1_PASSWORD=app_password_1
WORDPRESS_1_ID=production
WORDPRESS_1_DEFAULT=true
WORDPRESS_1_ALIASES=prod,main

# Site 2 (Staging)
WORDPRESS_2_URL=https://staging-site.com
WORDPRESS_2_USERNAME=admin
WORDPRESS_2_PASSWORD=app_password_2
WORDPRESS_2_ID=staging
WORDPRESS_2_ALIASES=stage,dev
```

The app password can be generated from WordPress admin panel following the [Application Passwords guide](https://make.wordpress.org/core/2020/11/05/application-passwords-integration-guide#Getting-Credentials).

## Architecture

### Core Components

1. **MCP Server (`src/server.ts`)**: 
   - Entry point that initializes the MCP server using the ModelContextProtocol SDK
   - Registers all WordPress tools with their handlers
   - Uses StdioServerTransport for communication with Claude Desktop
   - Validates environment variables and establishes WordPress connection on startup

2. **Site Manager (`src/config/site-manager.ts`)**:
   - Manages multiple WordPress site configurations
   - Lazy loads site configurations from environment variables
   - Maintains separate authenticated Axios clients for each site
   - Provides site detection from context (domain mentions, aliases, site IDs)
   - Supports both numbered multi-site config and legacy single-site config

3. **WordPress Client (`src/wordpress.ts`)**:
   - Manages authenticated Axios instance for WordPress REST API calls
   - Integrates with SiteManager for multi-site support
   - Handles authentication using Basic Auth with application passwords
   - Provides `makeWordPressRequest()` wrapper for all API calls with optional `siteId` parameter
   - Includes logging to `logs/wordpress-api.log` for debugging
   - Special handler `searchWordPressPluginRepository()` for WordPress.org plugin search

4. **Tool System (`src/tools/`)**: 
   - Each WordPress entity (posts, pages, media, etc.) has its own module
   - Each module exports tools array and handlers object
   - Tools use Zod schemas for input validation and type safety
   - All tools support optional `site_id` parameter for multi-site support
   - All tools are aggregated in `src/tools/index.ts`

### Tool Pattern

Each tool module follows this pattern:
```typescript
// Define Zod schemas for input validation
const listSchema = z.object({...});
const getSchema = z.object({...});
const createSchema = z.object({...});
const updateSchema = z.object({...});
const deleteSchema = z.object({...});

// Export tools array with MCP tool definitions
export const entityTools: Tool[] = [
  { name: "list_entity", description: "...", inputSchema: {...} },
  { name: "get_entity", description: "...", inputSchema: {...} },
  { name: "create_entity", description: "...", inputSchema: {...} },
  { name: "update_entity", description: "...", inputSchema: {...} },
  { name: "delete_entity", description: "...", inputSchema: {...} }
];

// Export handlers object with async functions
export const entityHandlers = {
  list_entity: async (params) => {...},
  get_entity: async (params) => {...},
  create_entity: async (params) => {...},
  update_entity: async (params) => {...},
  delete_entity: async (params) => {...}
};
```

### Unified Tool Architecture

The MCP server uses a **unified tool approach** to reduce complexity and tool count from ~65 to ~35 tools. Instead of separate tools for posts, pages, and custom post types, there are now unified tools that handle all content types.

#### **Unified Content Tools** (`unified-content.ts`) - 8 tools
Handles ALL content types (posts, pages, custom post types) with a single set of tools:
- `list_content` - List any content type with filtering and pagination
- `get_content` - Get specific content by ID and type
- `create_content` - Create new content of any type
- `update_content` - Update existing content of any type
- `delete_content` - Delete content of any type
- `discover_content_types` - Find all available content types
- `find_content_by_url` - Smart URL resolver with optional update
- `get_content_by_slug` - Search by slug across content types

#### **Unified Taxonomy Tools** (`unified-taxonomies.ts`) - 8 tools
Handles ALL taxonomies (categories, tags, custom taxonomies) with a single set of tools:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InstaWP/mcp-wp](https://github.com/InstaWP/mcp-wp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
