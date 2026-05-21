---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
npm install              # Install dependencies
npm run build            # Build with tsup
npm run dev              # Run ElizaOS in dev mode
npm start                # Run ElizaOS

# Code quality
npm run lint             # Format code with Prettier
npm run format           # Same as lint
npm run format:check     # Check formatting without changes
```

### Testing
```bash
npm test                 # Run all tests (component + e2e)
npm run test:component   # Vitest unit tests only
npm run test:e2e         # ElizaOS integration tests only
npm run test:coverage    # Coverage report with Vitest
npm run test:watch       # Watch mode for tests

# Run specific test file
npx vitest run __tests__/actions.test.ts
npx vitest run --reporter=verbose  # Detailed output
```

## Architecture

### Plugin Structure
```
plugin-polymarket/
├── src/
│   ├── actions/         # 16 trading actions
│   │   ├── placeOrder.ts
│   │   ├── sellOrder.ts
│   │   ├── redeemWinnings.ts
│   │   ├── redeemWinningsEnhanced.ts
│   │   ├── searchMarkets.ts
│   │   ├── explainMarket.ts
│   │   ├── getMarketPrice.ts
│   │   ├── getPortfolioPositions.ts
│   │   ├── getWalletBalance.ts
│   │   ├── depositUSDC.ts
│   │   ├── approveUSDC.ts
│   │   ├── setupTrading.ts
│   │   ├── getOrderBookSummary.ts
│   │   ├── syncMarkets.ts
│   │   ├── showFavoriteMarkets.ts
│   │   └── getAccountAccessStatus.ts
│   ├── services/        # Background services
│   │   ├── MarketSyncService.ts    # 24-hour market sync
│   │   ├── MarketDetailService.ts  # Market info fetching
│   │   └── RedemptionService.ts    # Auto-redemption
│   ├── providers/       # Context providers
│   │   └── marketDataProvider.ts   # Market context injection
│   ├── db/             # Database layer
│   │   ├── schema.ts   # Drizzle ORM schemas
│   │   └── queries.ts  # Database operations
│   ├── utils/          # Utilities
│   │   ├── clob.ts     # CLOB API client
│   │   ├── wallet.ts   # Wallet operations
│   │   └── market.ts   # Market helpers
│   ├── templates/      # LLM prompt templates
│   ├── types.ts        # TypeScript definitions
│   ├── plugin.ts       # Plugin configuration
│   └── index.ts        # Public exports
└── __tests__/          # Test suite
    ├── actions.test.ts
    ├── plugin.test.ts
    ├── provider.test.ts
    └── mocks/
```

### Action Pattern
Each action implements the `IAction` interface from ElizaOS:
```typescript
interface IAction {
  name: string;
  description: string;
  validate: (runtime: IAgentRuntime, message: Memory) => Promise<boolean>;
  handler: (runtime: IAgentRuntime, message: Memory, state?: State) => Promise<boolean>;
  examples: Array<{ user: string; content: { text: string } }>;
}
```

### Service Pattern
Services extend ElizaOS `Service` class:
```typescript
class MarketSyncService extends Service {
  async initialize(runtime: IAgentRuntime): Promise<void>
  async start(): Promise<void>
  async stop(): Promise<void>
}
```

### Provider Pattern
Providers implement `IProvider` interface:
```typescript
interface IProvider {
  get: (runtime: IAgentRuntime, message: Memory) => Promise<string>;
}
```

## Key Implementation Details

### CLOB API Integration
- Uses `@polymarket/clob-client` for order management
- Automatic credential derivation from private key
- L1 and L2 authentication handled transparently

### Market Data Management
- Local PGLite database for market caching
- Drizzle ORM for type-safe queries
- 24-hour background sync of 1000+ markets
- Efficient search with local indexing

### Trading Flow
1. **Setup**: `approveUSDC` → `setupTrading`
2. **Discovery**: `searchMarkets` → `explainMarket`
3. **Analysis**: `getMarketPrice` → `getOrderBookSummary`
4. **Execution**: `placeOrder` or `sellOrder`
5. **Management**: `getPortfolioPositions` → `redeemWinnings`

### Error Handling
- Comprehensive validation in each action
- Graceful degradation on API failures
- Detailed error messages for debugging
- Automatic retry logic for network issues

## Environment Configuration

### Required
```env
WALLET_PRIVATE_KEY=0x...  # Ethereum private key for trading
```

### Optional
```env
CLOB_API_URL=https://clob.polymarket.com  # CLOB endpoint (default)
CLOB_API_KEY=...          # Optional L2 API key
PGLITE_DATA_DIR=./.eliza/.elizadb  # Database location
```

## Testing Strategy

### Unit Tests (Vitest)
- Mock runtime and dependencies
- Test action validation logic
- Verify handler responses
- Coverage target: 80%+

### Integration Tests (ElizaOS)
- Full plugin initialization
- End-to-end action flows
- Service lifecycle testing
- Database operations

### Test Utilities
```typescript
// Common test setup in __tests__/setup/
createMockRuntime()       // Mock IAgentRuntime
createTestMessage()       // Mock Memory object
setupTestDatabase()       // In-memory PGLite
```

## Development Workflow

### Adding a New Action
1. Create action file in `src/actions/`
2. Implement IAction interface
3. Add to exports in `src/index.ts`
4. Write tests in `__tests__/`
5. Update README documentation

### Modifying Database Schema
1. Update `src/db/schema.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Okay-Bet/plugin-polymarket](https://github.com/Okay-Bet/plugin-polymarket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
