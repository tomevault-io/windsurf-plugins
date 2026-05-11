---
trigger: always_on
description: General Guidelines
---

# TypeScript Rules

## Coding Style

- Use clear lowerCamelCase or UpperCamelCase names for functions and variables, per
  usual TypeScript conventions.

- DO NOT use fully uppercase abbreviations: Use names like `mapHistoryToLlmMessages`. DO
  NOT use names like `mapHistoryToLLMMessages`.

- DO NOT use underscore prefixes for variables that are actually used.
  Underscore prefixes should only be used for genuinely unused parameters (like
  framework callbacks).

## Docstrings

- All major functions and types should have a *concise* docstring explaining their
  purpose. They should use `\**` … `*/` style comments.

  - Focus on any rationale or purpose.

  - Do NOT state obvious things about the code.

  This should cover

  - Public types

  - Major functions

  - Convex schemas, functions, actions, mutations, and queries

  It should NOT cover:

  - Test functions

  - Trivial internal helper functions

  Example:

  ```ts
  /**
   * Render a ContextSummary as readable markdown for both LLMs and users.
   */
  export function formatContextMarkdown(
    summary: ContextSummary,
    options?: { maxHoldings?: number },
  ): string {
    ...
  }
  ```

- **Document fields in type definitions, not at usage sites.** Place documentation
  directly on type/interface fields as the single source of truth.
  Reference the type documentation elsewhere using `@see TypeName.fieldName`.

  ```ts
  // GOOD: Documentation on type definition
  interface RunConfig {
    /** When true, logs full LLM request/response payloads for debugging. */
    logLlmCalls: boolean;
    /**
     * TEST ONLY: Disables automatic scheduling of backtest steps.
     * NEVER use in production.
     */
    testSkipScheduling?: boolean;
  }
  
  // Reference it elsewhere
  export const runConfigValidator = v.object({
    logLlmCalls: v.optional(v.boolean()),
    /** @see RunConfig.testSkipScheduling for documentation */
    testSkipScheduling: v.optional(v.boolean()),
  });
  
  // BAD: Documentation duplicated at multiple usage sites
  export const runConfigValidator = v.object({
    /** When true, logs full LLM request/response payloads for debugging. */
    logLlmCalls: v.optional(v.boolean()),
    /** TEST ONLY: Disables automatic scheduling... */
    testSkipScheduling: v.optional(v.boolean()),
  });
  ```

## Type Annotations

- Don’t use `any` to types unless absolutely necessary!
  Do not add `any` types to get type checking to pass.
  Use more precise types instead.
  Then make sure type checking passes.

- Avoid `as any` and unsafe casts.
  Prefer overloads or precise types at boundaries.

  ```ts
  // BAD: Silences type safety
  const logger = createAgentLogger(ctx, agentCtx as any);
  
  // GOOD: Provide a precise input shape or overload that matches
  const logger = createAgentLogger(ctx, {
    runId: runId as Id<'runs'>,
    agentId: agentId as Id<'agents'>,
    conversationId: conversationId as Id<'conversations'>,
    experimentRunId: experimentRunId,
  });
  // Or define overloads to accept both Id<> and string shared types, and narrow internally.
  ```

- **Extract and name inline object types.** DO NOT use anonymous inline types for
  complex structures that appear in multiple places.
  Create named types in shared locations.

  ```ts
  // BAD: Inline anonymous type duplicated across functions
  interface ExecutionResults {
    tradesSummary: {
      totalTrades: number;
      successfulTrades: number;
      trades: { symbol: string; action: 'buy' | 'sell'; price: number }[];
    };
  }
  
  // GOOD: Named type in shared location
  interface FullTradeSummary {
    stats: TradeSummaryStats;
    trades: TradeDetail[];
  }
  interface ExecutionResults {
    tradesSummary: FullTradeSummary;
  }
  ```

- **Consolidate duplicate calculation logic.** DO NOT duplicate calculations of related
  metrics. Create a single function that computes all related values together.

  ```ts
  // BAD: Same calculations scattered across files
  const totalBuyValue = trades
    .filter((t) => t.action === 'buy')
    .reduce((sum, t) => sum + t.value, 0);
  const totalSellValue = trades
    .filter((t) => t.action === 'sell')
    .reduce((sum, t) => sum + t.value, 0);
  
  // GOOD: Single shared function computes all related metrics
  function computeTradeSummaryStats(trades: Trade[]): TradeSummaryStats {
    return {
      totalBuyValue: trades.filter((t) => t.action === 'buy').reduce((sum, t) => sum + t.value, 0),
      totalSellValue: trades
        .filter((t) => t.action === 'sell')
        .reduce((sum, t) => sum + t.value, 0),
      uniqueTickers: new Set(trades.map((t) => t.symbol)).size,
    };
  }
  ```

## Exhaustiveness Checks

- **Always add exhaustiveness checks to `switch` statements on discriminated union
  types.** When switching on unions (like `field.kind` or `action.type`), include a
  `default` branch that assigns to `never`. This forces a compile-time error if a new
  variant is added but not handled.

  ```ts
  // GOOD: Exhaustiveness check catches missing cases at compile time
  switch (field.kind) {
    case 'string':
      return handleString(field);
    case 'number':
      return handleNumber(field);
    // ... all cases ...
    default: {
      const _exhaustive: never = field;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jlevy/tryscript](https://github.com/jlevy/tryscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
