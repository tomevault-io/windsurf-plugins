---
trigger: always_on
description: - Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
---

# CLAUDE.md

## Workflow & Behavioral Standards

### Plan Before Building

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- One task per subagent for focused execution

### Verification Before Done

- Never mark a task complete without proving it works
- Run tests, check compilation, demonstrate correctness
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"

### Autonomous Problem Solving

- When given a bug report: just fix it — don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Go fix failing CI tests without being told how

### Core Principles

- **Simplicity First**: Make every change as simple as possible. Minimal code impact.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
- **Demand Elegance**: For non-trivial changes, pause and ask "is there a more elegant way?" Skip this for simple, obvious fixes.

**Classify first:** Before any work, determine if it's a construct (`use-cases/`), example (`examples/`), or other. Read the matching guide from `.kiro/steering/` before proceeding.

---

## Project Overview

**AppMod Catalog Blueprints** — a comprehensive library of use case-driven AWS CDK L3 constructs for accelerating serverless development and modernization on AWS.

- **Package**: `@cdklabs/cdk-appmod-catalog-blueprints`
- **License**: Apache-2.0
- **Language**: TypeScript (source), multi-language via JSII (Python, Java, .NET)
- **Framework**: AWS CDK v2 (^2.218.0)
- **Build Tool**: Projen (generates package.json, tsconfig — do NOT edit these manually)
- **Node.js**: >= 18.12.0

## Critical Distinction: Constructs vs Examples

```
CONSTRUCTS (use-cases/)              EXAMPLES (examples/)
├─ Reusable library components       ├─ Deployable applications
├─ Abstract & extensible (OOP)       ├─ Concrete & opinionated
├─ Published to npm via JSII         ├─ Not published
├─ Must have unit + CDK Nag tests    ├─ Demonstrate usage of constructs
└─ Exported in use-cases/index.ts    └─ Include README with deploy steps
```

Never implement reusable construct logic inside example stacks. Always classify work first.

## Repository Structure

```
use-cases/                    # Source TypeScript (compiles to lib/)
├── framework/                # Core AI agent framework
│   ├── agents/               # BaseAgent, BatchAgent, InteractiveAgent
│   │   └── knowledge-base/   # IKnowledgeBase, BedrockKnowledgeBase
│   ├── foundation/           # Network (VPC), EventBridge, AccessLog
│   ├── bedrock/              # Bedrock model utils and IAM
│   └── custom-resource/      # Runtime definitions
├── document-processing/      # Document processing workflows
│   ├── base-document-processing.ts      # Layer 1: Abstract base
│   ├── bedrock-document-processing.ts   # Layer 2: Bedrock impl
│   ├── agentic-document-processing.ts   # Layer 3: Agent-powered
│   └── adapter/              # IAdapter, QueuedS3Adapter
├── webapp/                   # CloudFront + S3 frontend hosting
└── utilities/                # Observability, data masking, IAM utils, test-utils

examples/                     # Ready-to-deploy example applications
lib/                          # Compiled output (generated — do not edit)
test/                         # Integration tests
website/                      # Documentation site
.projenrc.ts                  # Project configuration (edit this, not package.json)
```

## Build & Test Commands

All commands use Projen-backed scripts. Never use ad hoc `tsc` or `jest` directly.

```bash
# Install
npm install

# Build (compile + test + lint + docgen)
npm run build

# Build variants
npm run build:fast          # Skip docgen
npm run build:no-test       # Skip tests (fastest iteration)

# Compile only
npm run compile

# Lint
npm run eslint

# Test — all
npm test

# Test — targeted
npm run test:document-processing:unit
npm run test:webapp:unit
npm run test:cdk-nag:all
npm run test:cdk-nag:document-processing
npm run test:cdk-nag:webapp
npm run test:security

# Test — specific file
npm test -- --testPathPattern="access-log"

# Test — watch mode
npm run test:watch

# After .projenrc.ts changes
npx projen
```

## Coding Standards

### TypeScript

- **Files**: kebab-case (`base-document-processing.ts`), tests: `{name}.test.ts`, CDK Nag: `{name}-nag.test.ts`
- **Indentation**: 2 spaces
- **Quotes**: Single quotes
- **Semicolons**: Required
- **Line length**: 150 characters max
- **Strict mode**: All strict flags enabled (`noImplicitAny`, `strictNullChecks`, etc.)
- **Imports**: Ordered — builtin → AWS CDK → constructs → relative, alphabetical within groups
- **Props**: Always `interface` with `readonly` properties, JSDoc on every property

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdklabs/cdk-appmod-catalog-blueprints](https://github.com/cdklabs/cdk-appmod-catalog-blueprints) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
