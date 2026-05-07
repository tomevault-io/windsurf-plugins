---
trigger: always_on
description: vibecheck is a vibe-themed CLI tool for running language model evaluations. This is the open source CLI tool (MIT licensed) that connects to the vibecheck API at vibescheck.io.
---

# vibecheck Project Guide

## Project Overview

vibecheck is a vibe-themed CLI tool for running language model evaluations. This is the open source CLI tool (MIT licensed) that connects to the vibecheck API at vibescheck.io.

**Get your API key at [vibescheck.io](https://vibescheck.io)**

## Architecture

This is a monorepo managed with npm workspaces:

- **@vibe/cli** (`packages/cli`) - The main CLI interface (open source)
- **@vibecheck/shared** (`packages/shared`) - Shared TypeScript types and Zod schemas

### Languages & Tech Stack

- **Language**: TypeScript 5.3+
- **Runtime**: Node.js 20+
- **CLI Framework**: Commander.js
- **Schema Validation**: Zod
- **CLI Styling**: chalk, ora
- **API**: vibecheck API at vibescheck.io

## Project Structure

```
vibecheck/
├── packages/                      # CLI packages
│   ├── cli/                       # CLI application (open source)
│   │   ├── src/
│   │   │   ├── index.ts           # Main entry point, commander setup
│   │   │   ├── commands/          # All CLI commands
│   │   │   │   ├── run.ts         # vibe check command
│   │   │   │   ├── suite.ts       # vibe set/get commands
│   │   │   │   ├── runs.ts        # vibe get runs command
│   │   │   │   ├── models.ts      # vibe get models command
│   │   │   │   ├── org.ts         # vibe get org command
│   │   │   │   └── redeem.ts      # vibe redeem command
│   │   │   └── utils/             # Utilities
│   │   │       ├── display.ts     # Display formatting
│   │   │       ├── config.ts      # Configuration management
│   │   │       └── auth-error.ts  # Authentication error handling
│   │   └── package.json           # Bins: vibe, vibes
│   │
│   └── shared/                    # Shared types & schemas (open source)
│       ├── src/
│       │   ├── types.ts           # TypeScript types
│       │   └── index.ts           # Exports
│       └── package.json
│
├── examples/                      # Example YAML eval files
│   ├── hello-world.yaml          # Basic checks example
│   ├── multilingual-pbj.yaml     # Multilingual testing
│   └── politics.yaml             # Political evaluation
├── scripts/                       # Publishing and build scripts
│   └── publish.sh                # npm publishing script
├── tests/                         # Test suites
│   ├── integration/              # Integration tests
│   ├── e2e/                      # End-to-end tests
│   ├── fixtures/                 # Test fixtures
│   └── helpers/                  # Test utilities
├── claude.md                     # This file
├── .cursorrules                  # Symlink to claude.md
├── README.md                     # Main documentation
├── CONTRIBUTING.md               # Development guidelines
└── package.json                  # Root workspace config
```

## Key Concepts

### Vibe-Themed Terminology

This project uses playful internet slang terminology:

**Command Structure:**
- `vibe check` (or `vibes check`) - Run evaluations
- `vibe set suite` - Save a suite
- `vibe get` - List or retrieve resources (suites, runs, models, org, vars)
- `vibe delete` - Delete resources (vars, secrets)
- `vibe redeem` - Redeem invite codes

**Results:**
Success rates are displayed with color coding:
- **Green** (>80% pass rate) - High success rate
- **Yellow** (50-80% pass rate) - Moderate success rate
- **Red** (<50% pass rate) - Low success rate

**Individual Conditionals:**
- ✅ **PASS** - Check passed
- ❌ **FAIL** - Check failed

### Evaluation Suite Format

Evaluation suites are defined in YAML:

```yaml
metadata:
  name: suite-name
  model: anthropic/claude-3.5-sonnet
  system_prompt: You are a helpful assistant  # optional
  mcp_server:  # optional
    url: "https://your-mcp-server.com"
    name: "server-name"
    authorization_token: "your-token"

evals:
  - prompt: Question to ask the model
    checks:
      - match: "*expected text*"  # glob pattern matching
      - not_match: "*unwanted text*"  # negated patterns
      - or:  # OR operator for multiple patterns
          - match: "*option1*"
          - match: "*option2*"
      - min_tokens: 10
      - max_tokens: 100
      - semantic:
          expected: "semantic target"
          threshold: 0.8
      - llm_judge:
          criteria: "what to judge"
```

### Check Types

1. **match** - Pattern matching supporting both glob and regex syntax
   - **Glob patterns** (recommended): `*hello*`, `goodbye*`, `*world`, `exact`
   - **Regex patterns** (advanced): `.*hello.*`, `^goodbye`, `world$`
2. **not_match** - Negated patterns (must NOT match)
3. **or** - OR operator for multiple patterns
4. **min_tokens**/**max_tokens** - Token length constraints
5. **semantic** - Compare semantic meaning using embeddings (local)
6. **llm_judge** - Use an LLM to judge the response quality

### Check Logic

**AND Logic (Array Format)**: Multiple checks in an array must ALL pass
```yaml
checks:
  - match: "*hello*"      # AND
  - min_tokens: 5         # AND
  - max_tokens: 100       # AND
```

**OR Logic (Explicit)**: Use the `or:` field when you want ANY of the patterns to pass
```yaml
checks:
  or:                   # OR (at least one must pass)
    - match: "*yes*"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hev/vibecheck](https://github.com/hev/vibecheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
