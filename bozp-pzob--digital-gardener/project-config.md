---
trigger: always_on
description: Generates Discord avatar URLs for all users without requiring API calls:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Digital Gardener is a modular TypeScript system that cultivates and curates content from multiple sources, enriching it with AI and organizing it through a plugin architecture.

## Common Commands

### Main Application
```bash
# Build and run production
npm run build && npm start

# Development mode
npm run dev

# Historical data collection
npm run historical

# Run with specific configuration
npm start -- --source=discord-raw.json --output=./custom-output

# Historical data for date range
npm run historical -- --source=elizaos.json --after=2024-01-10 --before=2024-01-16

# Channel discovery
npm run discover-channels

# Update configs from checklist
npm run update-configs

# User identity workflow (recommended order)
npm run build-user-index                                # 1. Build global user index
npm run fetch-avatars -- --update-index                 # 2. Generate avatar URLs and update index
npm run enrich-nicknames -- --all --use-index          # 3. Enrich all JSONs from index (fast!)

# Alternative: Enrich without building index first (slower, queries DB each time)
npm run enrich-nicknames -- --date=2026-01-12           # Single date from DB
npm run enrich-nicknames -- --from=2026-01-01 --to=2026-01-12  # Date range from DB
npm run enrich-nicknames -- --all                       # All JSON files from DB
npm run enrich-nicknames -- --all --dry-run            # Preview without writing

# Build/rebuild user index
npm run build-user-index                                # Generate data/discord/user-index.json
npm run build-user-index -- --output=./custom.json     # Custom output path
npm run build-user-index -- --dry-run                  # Preview without writing
```

### HTML Frontend (in html/ directory)
```bash
# Development server
npm run dev

# Build for production
npm run build

# Type checking
npm run check

# Database operations
npm run db:push
```

### AutoDoc (in autodoc/ directory)
```bash
# Generate documentation
npm run autodoc

# Development mode
npm run autodoc:dev

# Formatting
npm run lint && npm run format
```

## Architecture

### Plugin System
The system uses five plugin types:
- **Sources** (`src/plugins/sources/`) - Data collection (Discord, GitHub, APIs)
- **AI Providers** (`src/plugins/ai/`) - OpenAI/OpenRouter integration
- **Enrichers** (`src/plugins/enrichers/`) - Content enhancement (topics, images)
- **Generators** (`src/plugins/generators/`) - Summary generation
- **Storage** (`src/plugins/storage/`) - SQLite with encryption

### Core Components
- **ContentAggregator** (`src/aggregator/ContentAggregator.ts`) - Main orchestration engine
- **HistoricalAggregator** (`src/aggregator/HistoricalAggregator.ts`) - Historical data processing
- **Types** (`src/types.ts`) - Comprehensive type definitions including plugin interfaces

### Configuration
JSON configuration files in `config/` directory:
- `sources.json` - Default configuration
- `elizaos2.json` - Unified ElizaOS configuration (Discord + GitHub + Codex analytics)
- `hyperfy-discord.json` - Specialized configuration for Hyperfy Discord

Each config contains: `settings`, `sources`, `ai`, `enrichers`, `storage`, `generators` arrays.

### Environment Variables
Required in `.env`: `DISCORD_TOKEN`, `DISCORD_GUILD_ID`, `OPENAI_API_KEY`, `USE_OPENROUTER`, `CODEX_API_KEY`

## Data Sources
- Discord (raw messages, channels, announcements)
- GitHub (stats, general data)
- Crypto analytics (Codex, CoinGecko, Solana)
- Generic REST APIs

## Development Structure
```
src/
├── aggregator/     # Core engines
├── plugins/        # All plugin implementations
├── helpers/        # Utilities (cache, config, date, file, prompt)
└── types.ts        # Type definitions
```

The system supports specialized modes: `--onlyFetch` (no AI processing), `--onlyGenerate` (process existing data), and configurable output directories.

## Channel Management System

Unified TypeScript CLI (`scripts/channels.ts`) for Discord channel discovery and management:

```bash
# Discovery & Analysis
npm run channels -- discover                # Fetch channels from Discord (or raw data if no token)
npm run channels -- analyze                 # Run LLM analysis on channels needing it
npm run channels -- analyze --all           # Re-analyze all channels
npm run channels -- analyze --channel=ID    # Analyze single channel
npm run channels -- propose                 # Generate PR markdown with config changes

# Query Commands
npm run channels -- list [--tracked|--active|--muted|--quiet]
npm run channels -- show <channelId>
npm run channels -- stats

# Management Commands
npm run channels -- track <channelId>
npm run channels -- untrack <channelId>
npm run channels -- mute <channelId>
npm run channels -- unmute <channelId>

# Registry Commands
npm run channels -- build-registry          # Backfill from discordRawData
```

### Workflow
```bash
npm run channels -- discover   # Fetch channels
npm run channels -- analyze    # Run LLM analysis (TRACK/MAYBE/SKIP)
npm run channels -- propose    # Generate PR markdown
```

### Data Storage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bozp-pzob/digital-gardener](https://github.com/bozp-pzob/digital-gardener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
