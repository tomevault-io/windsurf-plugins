---
trigger: always_on
description: **slimcontext** is a lightweight, model-agnostic chat history compression library for AI assistants. It provides simple strategies to keep conversations concise while preserving context using a "Bring Your Own Model" (BYOM) approach.
---

# Copilot Instructions for slimcontext

## Repository Overview

**slimcontext** is a lightweight, model-agnostic chat history compression library for AI assistants. It provides simple strategies to keep conversations concise while preserving context using a "Bring Your Own Model" (BYOM) approach.

### High-Level Repository Information

- **Type**: TypeScript npm library/package
- **Languages**: TypeScript (primary), JavaScript (compiled output)
- **Target Runtime**: Node.js (CommonJS modules)
- **Framework**: Model-agnostic core; optional adapters (LangChain)
- **Package Manager**: pnpm (preferred) or npm (fallback)
- **Testing**: vitest
- **Linting**: ESLint with TypeScript support
- **Formatting**: Prettier

## Build Instructions and Development Workflow

### Prerequisites and Environment Setup

- **Node.js**: Version 20+ (recommended)
- **Package Manager**: pnpm 10.14.0 (preferred) or npm (fallback)

### Critical Build Steps (Always Follow This Order)

1. **Install Dependencies**

   ```bash
   # Preferred method (if pnpm available):
   pnpm install --frozen-lockfile

   # Fallback method (always works):
   npm install
   ```

   **Always run install before any other commands.** The `prepare` script automatically runs build after install.

2. **Build the Project**

   ```bash
   pnpm run build
   # Compiles TypeScript to dist/ directory
   ```

3. **Run Tests**

   ```bash
   pnpm run test
   # Runs all vitest tests
   # All tests should pass
   ```

4. **Format Code**

   ```bash
   pnpm run format        # Auto-format code
   pnpm run format:check  # Check formatting without changes
   ```

5. **Lint Code (Known Issue)**
   ```bash
   pnpm run lint
   ```

### Complete Development Workflow

```bash
# Clean start (recommended for agents):
rm -rf node_modules dist
pnpm install           # Always use pnpm for reliability
pnpm run test         # Verify tests pass
pnpm run format:check # Verify formatting
pnpm run build        # Final build
```

### CI/CD Pipeline Validation

The repository uses GitHub Actions CI that runs:

1. `pnpm install --frozen-lockfile`
2. `pnpm run lint`
3. `pnpm run format:check`
4. `pnpm run build`
5. `pnpm test`

**Note**: CI works because it runs in a different environment where the ESLint path issue doesn't occur.

## Project Layout and Architecture

### Core Directory Structure

```
/
├── src/                           # TypeScript source code
│   ├── index.ts                  # Main exports (trim, summarize, interfaces, adapters namespace)
│   ├── interfaces.ts             # Core type definitions (SlimContextMessage, etc.)
│   ├── adapters/                 # Integration adapters (optional)
│   │   └── langchain.ts          # LangChain adapter + helpers (compressLangChainHistory, toSlimModel)
│   └── strategies/               # Compression strategy implementations
│       ├── common.ts             # Shared token-budget utilities & defaults (thresholds, estimator)
│       ├── trim.ts               # TrimCompressor: token-threshold trimming (preserve system + recent)
│       └── summarize.ts          # SummarizeCompressor: token-threshold summarization (inject summary)
├── tests/                        # vitest test files
│   ├── trim.test.ts             # Tests for TrimCompressor
│   ├── summarize.test.ts        # Tests for SummarizeCompressor
│   └── langchain.test.ts        # Tests for LangChain adapter + helper
├── examples/                    # Documentation-only examples (not code)
│   ├── OPENAI_EXAMPLE.md        # Copy-paste OpenAI integration
│   ├── LANGCHAIN_EXAMPLE.md     # Copy-paste LangChain integration
│   └── LANGCHAIN_COMPRESS_HISTORY.md # One-call compressLangChainHistory usage
├── dist/                        # Compiled JavaScript output (generated)
└── package.json                # npm package configuration
```

### Configuration Files

- **tsconfig.json**: TypeScript compiler configuration (target: ES2019, CommonJS)
- **tsconfig.eslint.json**: Extended config for ESLint (includes tests)
- **.eslintrc.json**: ESLint configuration with TypeScript, import rules
- **.prettierrc.json**: Code formatting rules (single quotes, trailing commas)
- **.github/workflows/ci.yml**: CI/CD pipeline definition

### Key Architecture Elements

**Core Interfaces** (src/interfaces.ts):

- `SlimContextMessage`: Standard message format with role ('system'|'user'|'assistant'|'tool'|'human') and content
- `SlimContextChatModel`: BYOM interface requiring only `invoke(messages) -> response`
- `SlimContextCompressor`: Strategy interface for compression implementations
- `TokenEstimator`: `(message) => number` callback used for model-agnostic token budgeting

**Compression Strategies**:

- Token-threshold based design using the model’s max token window and a configurable threshold (default 70%).
- Shared config shape (TokenBudgetConfig): `{ maxModelTokens?, thresholdPercent?, estimateTokens?, minRecentMessages? }`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentailor/slimcontext](https://github.com/agentailor/slimcontext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
