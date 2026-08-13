---
trigger: always_on
description: Azure Functions v2 timer-triggered app for DEFRA MMO (Marine Management Organisation). Replaces CRON jobs to orchestrate scheduled tasks for the `mmo-fes-reference-data-reader` service and reconcile certificate data with a business continuity system.
---

# MMO FES Function App - AI Coding Agent Instructions

## Project Overview
Azure Functions v2 timer-triggered app for DEFRA MMO (Marine Management Organisation). Replaces CRON jobs to orchestrate scheduled tasks for the `mmo-fes-reference-data-reader` service and reconcile certificate data with a business continuity system.

**Two Azure Functions:**
- `mmo-fes-functionapp`: Timer-triggered HTTP orchestrator with retry logic (calls data reader every X hours)
- `mmo-fes-reconciliationapp`: HTTP-triggered MongoDB → Business Continuity API reconciliation

## Architecture & Patterns

### Retry Pattern with Exponential Backoff
Both functions implement custom retry logic with increasing delays:
```javascript
// Pattern: retry(fn, retries, delayFn) where delay = (totalRetries - retriesRemaining) * baseDelay
// Example: 5-min base delay → attempts at 0min, 0min, 5min, 10min, 15min
const calcDelay = (delay, totalRetries) => (retriesRemaining) =>
  (totalRetries - retriesRemaining) * delay;
```

### Config via Environment Variables
All functions load config from env vars with fallback defaults in the module-level `config` object. Use `overrideConfig` parameter in tests:
```javascript
await func(context, myTimer, { retries: 2, retryDelay: 1000 });
```

### Application Insights Integration
Custom correlation with `ai.operation.id` from Azure context. Always initialize AppInsights first, then axios interceptors:
```javascript
appInsights.init(config.instrumentationKey, context);
axiosInterceptors.init(axios);  // Adds duration tracking to responses
```

### HTTPS Agent with Custom CA Bundle
Loads `cabundle.pem` from function directory for custom certificate chains. Located at `${context.executionContext.functionDirectory}/../cabundle.pem`.

## Development Workflow

### Local Development
```bash
# Prerequisites: Azure Functions Core Tools v3+, Azurite storage emulator
npm install
azurite  # Run in separate terminal
func start  # Starts both functions
```

### Testing
```bash
npm test              # Run tests with coverage
npm run test:ci       # CI mode with jest-junit reporter
```

**Test Conventions:**
- Use `jest.spyOn()` for mocking, never reassign `require()`
- Mock MongoDB via `__mocks__/mongodb.js` (configured in `package.json` moduleNameMapper)
- Mock `setTimeout` to execute callbacks immediately: `jest.spyOn(global, 'setTimeout').mockImplementation((callback) => callback())`
- Context structure: `{ log: jest.fn(), executionContext: { functionDirectory: __dirname }, traceContext: {}, operationId: '' }`

**Coverage Requirements (enforced):**
- Branches: 90%, Functions: 90%, Lines: 90%, Statements: 90%

## Project-Specific Conventions

### Logging Format
Structured logs with bracketed tags:
```javascript
context.log('[SCHEDULED-JOBS][LANDING-AND-REPORTING][STARTED]', timeNow());
context.log(`[SCHEDULED-JOBS][BC-RECONCILIATION][CONFIG][url: ${url}]`, timeNow());
```

### Function Structure
Every function exports a single async function with signature:
```javascript
const func = async (context, myTimer, overrideConfig) => {
  // 1. Log start, merge overrideConfig
  // 2. Check myTimer.IsPastDue
  // 3. Initialize AppInsights + interceptors
  // 4. Execute main logic with retry pattern
  // 5. Track events and log completion
};
module.exports = func;
```

### Timer vs HTTP Triggers
- `mmo-fes-functionapp`: Timer trigger (`timerTrigger` binding) with `CRONTIME` env var
- `mmo-fes-reconciliationapp`: HTTP trigger (`httpTrigger` + `http` output binding)

## Key Files & Directories

- `mmo-fes-functionapp/index.js`: Main timer function with HTTP retry orchestration
- `mmo-fes-reconciliationapp/index.js`: MongoDB → BC API reconciliation with batch processing
- `src/appInsights.js`: Shared AppInsights wrapper with operation correlation
- `src/axiosInterceptors.js`: Request/response duration tracking using `perf_hooks`
- `__mocks__/mongodb.js`: Jest mock for MongoDB (returns COMPLETE/VOID certificate fixtures)
- `function.json`: Azure Functions binding definitions (timer schedule, HTTP triggers)
- `host.json`: Defines both functions in `functions` array, enables AppInsights live metrics

## CI/CD & Deployment

### GitFlow Branching
**Strictly enforced** - pipelines fail on non-standard branch names:
- `main`: Production releases
- `develop`: Integration branch
- `feature/*`, `epic/*`: Feature development
- `hotfix/*`: Production fixes

### Azure Pipelines
Uses shared template from `mmo-fes-pipeline-common` repo:
```yaml
# azure-pipelines.yml extends shared template
parameters:
  - deployFromFeature: false  # Override to deploy feature branches
  - skipPRE1: false            # Skip PRE1 environment
```

### Docker Build
Node 22 runtime on Azure Functions v4:
```dockerfile
FROM mcr.microsoft.com/azure-functions/node:4-node24
# Includes githash tracking via ARG GIT_HASH
```

## Common Pitfalls

1. **MongoDB Mock Not Applied**: Ensure `moduleNameMapper` in `package.json` points to `__mocks__/mongodb.js`
2. **Axios Import Path**: Use `axios/dist/node/axios.cjs` in Jest moduleNameMapper to avoid ESM issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DEFRA/eutd-mmo-fes-function-app](https://github.com/DEFRA/eutd-mmo-fes-function-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
