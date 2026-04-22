---
trigger: always_on
description: **Contributor Report** is a GitHub Action that evaluates PR contributor quality using objective GitHub metrics to combat
---

# Development Guide

## Project Overview

**Contributor Report** is a GitHub Action that evaluates PR contributor quality using objective GitHub metrics to combat
AI-generated spam PRs (also known as "AI slop" or "slop code"). It analyzes a contributor's GitHub activity history and
calculates scores based on PR merge rate, contributions to quality repositories, community engagement, and behavioral
patterns.

The goal is to help open source maintainers identify low-quality, spam, or AI-generated contributions that waste
maintainer time and resources, while being fair to legitimate contributors, especially newcomers.

## Common Commands

```bash
pnpm install          # Install dependencies
pnpm test             # Run tests
pnpm lint             # Run ESLint
pnpm bundle           # Format + package (run after changing src/)
pnpm run all          # Format, lint, test, coverage, and package
pnpm local-action     # Test action locally (requires .env file)
```

To run a single test file:

```bash
NODE_OPTIONS=--experimental-vm-modules NODE_NO_WARNINGS=1 pnpm exec jest __tests__/metrics/pr-history.test.ts
```

## Architecture

```text
src/
├── index.ts              # Entry point
├── main.ts               # Main action orchestration
├── types/                # TypeScript interfaces
│   ├── index.ts          # Re-exports all types
│   ├── config.ts         # Configuration types and defaults (MetricThresholds, FailAction, etc.)
│   ├── metrics.ts        # Metric data structures (10 data interfaces + AllMetricsData)
│   ├── scoring.ts        # Scoring result types (AnalysisResult, ActionOutput, ANALYSIS_CONSTANTS)
│   └── github.ts         # GitHub API response types (GraphQLContributorData, PRContext)
├── config/               # Input parsing and defaults
│   ├── index.ts          # Re-exports config functions
│   ├── inputs.ts         # Parse action inputs from env/action.yml
│   └── defaults.ts       # Default config, validation, VALID_METRIC_NAMES
├── api/                  # GitHub API client
│   ├── index.ts          # Re-exports API functions
│   ├── client.ts         # GitHubClient class (fetch, comment, label, PR context)
│   ├── queries.ts        # GraphQL queries (contributor data, org membership, rate limit)
│   └── rate-limit.ts     # Rate limit handling with exponential backoff (max 3 retries)
├── metrics/              # Individual metric calculators (extract + check pattern)
│   ├── index.ts          # Re-exports all metric functions
│   ├── pr-history.ts     # PR merge rate analysis
│   ├── repo-quality.ts   # Contributions to starred repos
│   ├── reactions.ts      # Comment reactions (positive + negative)
│   ├── account-age.ts    # Account age and activity consistency
│   ├── issue-engagement.ts # Issue engagement metrics
│   ├── code-review.ts    # Code review contributions
│   ├── merger-diversity.ts # Unique maintainers who merged PRs
│   ├── repo-history.ts   # Track record in specific repository
│   ├── profile-completeness.ts # GitHub profile richness (0-100 score)
│   └── suspicious-patterns.ts # Cross-metric spam detection (4 pattern types)
├── scoring/              # Score calculation
│   ├── index.ts          # Re-exports scoring functions
│   └── engine.ts         # Main scoring aggregation and evaluation
└── output/               # Output formatting
    ├── index.ts          # Re-exports output functions
    ├── comment.ts        # PR comment generation (full/passed/whitelist + verbose details)
    └── formatter.ts      # Action outputs, console ASCII table, and job summary
```

### Test & Fixture Structure

```text
__tests__/                # Test files (ESM with jest.unstable_mockModule)
├── api/
│   └── rate-limit.test.ts
├── config/
│   └── inputs.test.ts
├── metrics/
│   ├── account-age.test.ts
│   ├── code-review.test.ts
│   ├── issue-engagement.test.ts
│   ├── pr-history.test.ts
│   ├── reactions.test.ts
│   └── repo-quality.test.ts
├── scoring/
│   └── engine.test.ts
└── output/
    ├── comment.test.ts
    └── formatter.test.ts

__fixtures__/             # Test fixtures and mocks
├── core.ts               # Mocked @actions/core functions
├── github.ts             # Mocked @actions/github context
├── testData.ts           # Sample GraphQL responses for tests
└── api-responses/
    └── contributor-data.ts # Full API response fixtures

docs/metrics/             # Detailed metric documentation (12 markdown files)
```

**Test coverage gap:** The newer metrics (merger-diversity, repo-history, profile-completeness, suspicious-patterns) are
tested indirectly through `scoring/engine.test.ts` but lack dedicated test files in `__tests__/metrics/`.

## Key Concepts

### Metric Pattern

Each metric follows a consistent pattern:

```typescript
// 1. Extract data from GraphQL response
function extractXxxData(data: GraphQLContributorData, ...params): XxxData

// 2. Check against threshold
function checkXxx(data: XxxData, threshold: number): MetricCheckResult
```

**Available Metrics (13 total — 12 configurable + 1 auto):**

- `prMergeRate` - PR merge rate analysis
- `repoQuality` - Contributions to starred repos
- `positiveReactions` / `negativeReactions` - Community engagement

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdiegosierra/contributor-report](https://github.com/jdiegosierra/contributor-report) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
