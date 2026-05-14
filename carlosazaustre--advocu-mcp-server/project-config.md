---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A unified MCP (Model Context Protocol) server that enables both Google Developer Experts (GDEs) and Microsoft MVPs to report their activities through AI-powered conversational interfaces. The server integrates with:
- **Advocu GDE API** for Google GDE activity submissions
- **Microsoft MVP API** for Microsoft MVP activity submissions

Both integrations are optional - users can configure one or both depending on their needs.

## Package Manager

This project uses **npm** (not pnpm or yarn). All commands should use npm.

## Common Commands

### Development
```bash
npm run dev          # Run server in development mode (uses tsx)
npm start            # Run compiled server from dist/
```

### Building
```bash
npm run build        # Compile TypeScript to dist/
```

### Linting and Formatting
```bash
npm run lint         # Check code with Biome
npm run format       # Format code with Biome (writes changes)
```

### Publishing
```bash
npm run prepublishOnly    # Runs before publishing (builds automatically)
npm run publish:dry       # Test publish without actually publishing
npm run release           # Create new release with standard-version
npm run release:patch     # Patch version bump
npm run release:minor     # Minor version bump
npm run release:major     # Major version bump
npm run postrelease       # Pushes tags and publishes to npm
```

## Architecture

### Entry Point and Server Flow

1. **index.ts** - Entry point that loads environment variables and instantiates the unified server
2. **unifiedServer.ts** - Main unified server class (`UnifiedActivityReportingServer`) that:
   - Initializes MCP server with stdio transport
   - Validates environment variables for both GDE and MVP (at least one must be configured)
   - Registers tools for enabled integrations:
     - 7 GDE activity tools (if `ADVOCU_ACCESS_TOKEN` is set)
     - 3+ MVP activity tools (if `MVP_ACCESS_TOKEN` and `MVP_USER_PROFILE_ID` are set)
   - Routes tool calls to the appropriate API (GDE or MVP)
3. **server.ts** - Legacy GDE-only server (kept for backwards compatibility)
4. **mvpServer.ts** - Standalone MVP server (can be used independently)

### Activity Type System

The codebase uses a type-safe, inheritance-based structure for activity drafts:

#### GDE Activity Types (in `src/interfaces/`)
- **ActivityDraftBase** - Base interface with common properties (title, description, activityDate, tags, additionalInfo, private)
- **Specific activity interfaces** extend the base and add unique properties:
  - `ContentCreationDraft` - For articles, videos, podcasts, etc.
  - `PublicSpeakingDraft` - For talks and presentations
  - `WorkshopDraft` - For training sessions
  - `MentoringDraft` - For mentoring activities
  - `ProductFeedbackDraft` - For product feedback submissions
  - `GooglerInteractionDraft` - For interactions with Google employees
  - `StoryDraft` - For success stories

#### MVP Activity Types (in `src/interfaces/mvp/`)
- **MVPActivityBase** - Base interface with common MVP properties (title, description, date, url, targetAudience, role, technologyFocusArea, etc.)
- **Specific MVP activity interfaces** extend the base and add metrics:
  - `MVPVideoActivity` - For videos, webinars, livestreams (liveStreamViews, onDemandViews, numberOfSessions)
  - `MVPBlogActivity` - For blog posts and articles (numberOfViews, subscriberBase)
  - `MVPSpeakingActivity` - For speaking engagements (inPersonAttendees, liveStreamViews, onDemandViews)
  - `MVPBookActivity` - For books and e-books (copiesSold, subscriberBase)

### Type Definitions

#### GDE Enums (in `src/types/`)
- `ContentType` - 8 types (Articles, Books, Videos, etc.)
- `EventFormat` - 3 formats (In-Person, Virtual, Hybrid)
- `Country` - 251 country codes
- `InteractionFormat` - 8 formats for Googler interactions
- `InteractionType` - 6 types of interactions
- `ProductFeedbackContentType` - 2 types
- `SignificanceType` - 6 types for stories

#### MVP Enums (in `src/types/mvp/`)
- `MVPActivityType` - 20+ types (Blog, Video, Speaking, Book, Mentorship, Open Source, etc.)
- `MVPActivityRole` - 9 roles (Author, Speaker, Host, Contributor, etc.)
- `MVPTargetAudience` - 6 audiences (Developer, Technical Decision Maker, Business Decision Maker, Student, IT Pro, End User)

### API Integration

#### GDE (Advocu) API
- Base URL: `https://api.advocu.com/personal-api/v1/gde` (configurable via `ADVOCU_API_URL`)
- Endpoints: `/activity-drafts/{activity-type}`
- Authentication: Bearer token via `ADVOCU_ACCESS_TOKEN`
- Rate limit: 30 requests per minute (handled with 429 error detection)

#### Microsoft MVP API
- Base URL: `https://mavenapi-prod.azurewebsites.net/api` (configurable via `MVP_API_URL`)
- Endpoints: `POST /Activities/`
- Authentication: Bearer token via `MVP_ACCESS_TOKEN`
- Additional requirement: `MVP_USER_PROFILE_ID` (your MVP profile ID number)
- Payload format: Wraps activity in `{ "activity": { ...fields } }`

### MCP Tools

#### GDE Tools (when `ADVOCU_ACCESS_TOKEN` is set)
1. `submit_gde_content_creation`
2. `submit_gde_public_speaking`
3. `submit_gde_workshop`
4. `submit_gde_mentoring`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlosazaustre/advocu-mcp-server](https://github.com/carlosazaustre/advocu-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
