---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Reddit MCP (Model Context Protocol) server that provides tools for interacting with the Reddit API. It's built with TypeScript and uses FastMCP to expose Reddit functionality as tools that can be used by AI assistants.

## Available Tools

### Read-only Tools (Client Credentials Only)

- `get_reddit_post` - Get a specific Reddit post with engagement analysis
- `get_top_posts` - Get top posts from a subreddit or home feed
- `get_user_info` - Get detailed information about a Reddit user
- `get_subreddit_info` - Get subreddit details, stats, and community insights
- `get_trending_subreddits` - Get currently trending/popular subreddits
- `search_reddit` - Search for posts across Reddit with filters
- `get_post_comments` - Get comments from a specific post with threading
- `get_user_posts` - Get posts submitted by a specific user
- `get_user_comments` - Get comments made by a specific user

### Write Tools (User Credentials Required)

**IMPORTANT**: These tools require both REDDIT_USERNAME and REDDIT_PASSWORD to be configured.

- `create_post` - Create a new post in a subreddit (text or link)
- `reply_to_post` - Post a reply to an existing Reddit post or comment
- `edit_post` - Edit your own Reddit post (self-text posts only, titles cannot be edited)
- `edit_comment` - Edit your own Reddit comment
- `delete_post` - **PERMANENTLY** delete your own Reddit post (cannot be undone!)
- `delete_comment` - **PERMANENTLY** delete your own Reddit comment (cannot be undone!)

### Server Modes

The server supports two transport modes:

1. **HTTP Server (Default)**: Runs on port 3000 with `/mcp` endpoint
   - Used for Docker deployments and direct execution
   - Access via: `http://localhost:3000/mcp`
   - SSE endpoint: `http://localhost:3000/sse`

2. **Stdio Mode**: For CLI and npx usage
   - Automatically enabled when using `npx reddit-mcp-server` or the bin entry point
   - Used for integration with Claude Desktop and other MCP clients

## Development Commands

```bash
# Install dependencies
pnpm install

# Build TypeScript to JavaScript with tsup
pnpm build

# Run the MCP inspector for development/testing
pnpm inspect

# Build and run inspector in one command
pnpm dev

# Build and start the server via npx
pnpm start

# Format code with Prettier
pnpm format

# Check code formatting
pnpm format:check

# Lint code with ESLint
pnpm lint

# Fix linting issues
pnpm lint:fix
```

## Architecture

### Core Components

1. **Reddit Client** (`src/client/reddit-client.ts`): Singleton pattern implementation that handles:
   - OAuth2 authentication (client credentials and password flow)
   - Automatic token refresh via axios interceptors
   - Rate limiting and error handling
   - Both read-only and authenticated operations

2. **Tool Modules** (`src/tools/`): Modular organization by functionality:
   - `post-tools.ts`: Post creation, retrieval, and management
   - `comment-tools.ts`: Comment retrieval and threading
   - `subreddit-tools.ts`: Subreddit info, statistics, trending
   - `user-tools.ts`: User information and engagement insights
   - `search-tools.ts`: Reddit search functionality

3. **Type Definitions** (`src/types.ts`): Comprehensive TypeScript types for all Reddit entities

### Authentication Flow

The server supports three authentication modes configured via `REDDIT_AUTH_MODE`:

1. **auto (default)**: Automatically chooses the best authentication method
   - If REDDIT_CLIENT_ID and REDDIT_CLIENT_SECRET are provided: Uses OAuth (60-100 req/min)
   - Otherwise: Falls back to anonymous mode (~10 req/min)
   - Gracefully degrades without failing

2. **authenticated**: Requires OAuth credentials
   - Requires REDDIT_CLIENT_ID and REDDIT_CLIENT_SECRET
   - Server fails to start if credentials are missing
   - Provides higher rate limits (60-100 req/min)
   - Use for production environments with guaranteed credentials

3. **anonymous**: Uses public JSON API without authentication
   - No credentials required - zero-setup experience
   - Lower rate limit (~10 req/min)
   - Perfect for testing and development
   - Read-only operations work without any Reddit app setup

**Write operations** (create_post, reply_to_post, edit_post, edit_comment, delete_post, delete_comment):

- Require REDDIT_USERNAME and REDDIT_PASSWORD in **any** mode
- Will fail gracefully with a clear error message if credentials are missing
- Token management is handled automatically by the Reddit client

### Safe Mode (Spam Protection)

The server includes optional safeguards to protect against Reddit's spam detection, configured via `REDDIT_SAFE_MODE`:

1. **off (default)**: No safeguards, original behavior
2. **standard**: Recommended for normal use
   - 2-second delay between write operations
   - Duplicate content detection (tracks last 10 items)
3. **strict**: For cautious automated posting
   - 5-second delay between write operations
   - Aggressive duplicate detection (tracks last 20 items)

**Features:**

- **Rate Limiting**: Enforces minimum delays between write operations to avoid spam flags

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanburke/reddit-mcp-server](https://github.com/jordanburke/reddit-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
