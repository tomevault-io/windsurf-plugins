---
trigger: always_on
description: This is a **Bun/TypeScript Twitter bot** for NFT artists on Tezos. It automates:
---

# AI Agent Instructions for SKLLZ Twitter Bot

## Project Overview

This is a **Bun/TypeScript Twitter bot** for NFT artists on Tezos. It automates:

1. **Thank-you tweets** when art sells (synced from objkt.com + TzKT APIs)
2. **Weekly shill threads** showcasing latest 5 artworks
3. Uses **OpenAI o3-mini** for AI-generated tweet content
4. Uses **PostgreSQL** for tracking sales, tokens, and dynamic schedules

### Supported NFT Types

The bot currently tracks and promotes three types of Tezos NFTs:

1. **Standard Tokens (1/1s)** - Unique pieces with active listings on objkt.com
   - Fetched via `getMyMintedTokens()` in `objkt.ts`
   - Includes editions with limited supply

2. **Open Editions** - Unlimited mints available for purchase
   - Fetched via `getActiveOpenEditions()` in `objkt.ts`
   - Tracks individual mints as sales events

3. **Generative Collections (editart)** - Algorithmic art collections
   - Fetched via `getGenerativeCollections()` in `objkt.ts`
   - Each minted token tracked individually
   - Supports editart-style generative projects

All three types sync to the same `tokens` table and generate thank-you tweets when sold/minted.

## Critical Architecture Points

### 1. Bun Runtime & ESM Modules

- **Always use ESM syntax**: `import`/`export`, not CommonJS
- Entry points use `import.meta.main` pattern (see `thank-you.ts`, `shill-thread.ts`)
- Scripts run via `bun run <file>` (no transpilation needed)
- Use `process.argv.includes('--flag')` for CLI args

### 2. Database-Driven Scheduling (Not Simple Cron)

- `src/index.ts` loads schedules from PostgreSQL `schedules` table
- Uses `croner` library to create dynamic jobs at runtime
- Schedules managed via CLI: `bun run schedule add/list/enable/disable/remove`
- **PostgreSQL advisory locks** (`utils/locks.ts`) prevent duplicate runs of same schedule
- Jobs call `executeSchedule()` which wraps `processThankYouTweets()` or `processShillThread()`

### 3. Dual Workflow Pattern: Direct CLI + Scheduled Execution

Every automation has 2 entry points:

- **Direct**: `bun run thank` or `bun run shill` (calls exported function)
- **Scheduled**: `src/index.ts` calls same exported function via schedule trigger
- Both support `--dry-run` flag for testing without posting tweets

### 4. Tezos NFT Data Pipeline

Data flows through 3 layers:

1. **Fetch** (`objkt.ts`): GraphQL queries to objkt.com
   - `getMyMintedTokens()` - 1/1s with active listings
   - `getActiveOpenEditions()` - open editions
   - `getGenerativeCollections()` - generative collections
   - `getListingSales()` / `getOpenEditionSales()` / `getGenerativeSales()` - sales data
2. **Transform** (`transformers.ts`): Convert GraphQL responses to flat structures
   - `transformMintedToken()`, `transformOpenEdition()`, `transformGenerativeCollection()`
   - `transformListingSale()`, `transformMintSale()` - generate sale records
3. **Sync** (`sync.ts`): Upsert to PostgreSQL `tokens` and `nft_sales` tables
   - `syncObjktData()` - called before every tweet automation run
   - Handles duplicates via `ON CONFLICT` clauses

### 5. OpenAI Integration (o3-mini)

- **Model requirement**: Use `o3-mini` (not gpt-4o) for reasoning capabilities
- **Token parameter**: Use `max_completion_tokens` (not `max_tokens`)
- All prompts centralized in `src/prompts.ts` with `system` and `user` messages
- Generates authentic, varied tweets (anti-spam pattern)

### 6. Twitter API v2 Usage

- Library: `twitter-api-v2` (OAuth 1.0a)
- Client created via `createTwitterClient()` in `index.ts`
- Thread posting pattern in `shill-thread.ts`:
  - Post intro tweet → capture `tweet.data.id`
  - Reply to previous tweet using `{ reply: { in_reply_to_tweet_id } }`
  - 2-3s random delay between tweets to avoid rate limits
  - Retry with exponential backoff (`retryWithBackoff()`)

## Key Conventions

### Configuration

- All env vars loaded via `config.ts` (uses `dotenv/config`)
- Validation: `getEnvVar()` throws if required var missing
- Wallet addresses are **comma-separated** in `WALLET_ADDRESSES` env var

### Database Access

- Single global `pool` exported from `db.ts`
- Always parameterize queries: `pool.query('SELECT * FROM tokens WHERE id = $1', [id])`
- Dry-run pattern: Log instead of executing when `dryRun = true`

### Logging

- Use `logger` from `utils/logger.ts` (not `console.log`)
- Emoji prefixes: `✅` success, `❌` errors, `⚠️` warnings, `🔍` dry-run, `🐦` tweets

### Error Handling

- Graceful shutdown hooks in `index.ts`: `SIGINT`/`SIGTERM` → release locks + close DB
- Unhandled rejections/exceptions logged before exit
- Retry logic with backoff for Twitter API calls

## Common Development Workflows

### Testing Changes

```bash
# Test API connections
bun run test

# Dry-run thank-you tweets (no posts)
bun run thank:dry-run

# Dry-run shill thread
bun run shill:dry-run

# Test data sync
bun run sync:dry-run
```

### Adding New Features

#### New Tweet Type

1. Add prompt templates to `src/prompts.ts`:
   ```typescript
   export const prompts = {
     myNewTweet: {
       system: `System prompt defining AI behavior...`,
       user: (param1, param2) => `User prompt with ${param1}...`,
     },
   };
   ```
2. Create new file `src/my-new-tweet.ts` following the dual-workflow pattern:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skullzarmy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
