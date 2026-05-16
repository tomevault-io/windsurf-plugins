---
trigger: always_on
description: Defines the development workflow (build, lint, test, migrate commands), coding rules (TDD authority, no scope creep, escalate unknowns), the full tech stack (Node.js, TypeScript strict, Express, PostgreSQL/pgvector, Redis/BullMQ, Vitest, Pino, Zod, web-tree-sitter, Octokit), and a directory of all spec files (phases/), architecture docs, and task breakdowns (tasks/).
---


# DocAlign — Implementation

A documentation-reality alignment engine that detects when repo documentation drifts from code reality, alerts developers on PRs, and serves verified docs to AI coding agents via MCP.

## Current Phase

IMPLEMENTATION. Planning is complete. All specs, TDDs, and task breakdowns are finalized.

## Commands

```bash
npm run build          # TypeScript compilation
npm run lint           # ESLint check
npm run lint:fix       # ESLint auto-fix
npm run format         # Prettier format
npm run typecheck      # tsc --noEmit
npm run test           # Vitest run (all tests)
npm run test:watch     # Vitest watch mode
npm run migrate:up     # Run database migrations
npm run migrate:down   # Rollback last migration
docker compose up -d   # Start PostgreSQL + Redis (local dev)
docker compose down    # Stop local services
```

**After every task:** run `npm run typecheck && npm run test` to verify. Do not move to the next task until both pass.

**After every file edit:** run `npm run lint:fix` on changed files.

## Rules

1. **Read the task file before starting.** Each task references specific TDD sections, types, and test cases. Read them.
2. **Read before writing.** Before modifying any file, read it and its relevant spec sections. Do not work from memory.
3. **Follow existing patterns.** Match the code style, error handling, and naming conventions established by prior tasks.
4. **TDD is the authority.** If the task file and TDD disagree, the TDD wins. If the TDD and the API contracts documentation disagree, escalate.
5. **All tests must pass.** `npm run typecheck && npm run test` must succeed after every task.
6. **No scope creep.** Implement exactly what the task specifies. No extra features, no premature abstractions.
7. **Escalate unknowns.** If a spec is ambiguous or contradictory, ask. Do not guess.
8. **Commit after each task.** One task = one commit. Use descriptive messages.

## Tech Stack

<!-- docalign:semantic id="semantic-tech-stack-typescript" claim="Runtime: Node.js + TypeScript (strict mode)" -->
- **Runtime:** Node.js + TypeScript (strict mode)
<!-- docalign:semantic id="semantic-tech-stack-express" claim="Server: Express.js" -->
- **Server:** Express.js
<!-- docalign:semantic id="semantic-tech-stack-postgresql" claim="Database: PostgreSQL (pgvector), SQLite (CLI mode via better-sqlite3)" -->
- **Database:** PostgreSQL (pgvector), SQLite (CLI mode via better-sqlite3)
<!-- docalign:semantic id="semantic-tech-stack-redis-bullmq" claim="Queue: Redis + BullMQ" -->
- **Queue:** Redis + BullMQ
<!-- docalign:semantic id="semantic-tech-stack-vitest" claim="Testing: Vitest" -->
- **Testing:** Vitest
<!-- docalign:semantic id="semantic-tech-stack-pino" claim="Logging: Pino (structured JSON)" -->
- **Logging:** Pino (structured JSON)
<!-- docalign:semantic id="semantic-tech-stack-zod" claim="Validation: Zod" -->
- **Validation:** Zod
<!-- docalign:semantic id="semantic-tech-stack-web-tree-sitter" claim="AST Parsing: web-tree-sitter (WASM)" -->
- **AST Parsing:** web-tree-sitter (WASM)
<!-- docalign:semantic id="semantic-tech-stack-octokit" claim="GitHub: Octokit (GitHub App auth)" -->
- **GitHub:** Octokit (GitHub App auth)

## Key Files

### Specs (implementation reference)

| File | Content |
|------|---------|
| `src/shared/types.ts` | Canonical TypeScript interfaces and Row types |
| `phases/tdd-0-codebase-index.md` | L0: AST parsing, entity indexing, lookup APIs |
| `phases/tdd-1-claim-extractor.md` | L1: Doc parsing, regex extractors, claim pipeline |
| `phases/tdd-2-mapper.md` | L2: Claim-to-code mapping (3-step progressive) |
| `phases/tdd-3-verifier.md` | L3: Deterministic verification (Tier 1-2) |
| `phases/tdd-4-triggers.md` | L4: Webhook handlers, scan queue, pipeline orchestration |
| `phases/tdd-5-reporter.md` | L5: PR comments, Check Runs, health scores |
| `phases/tdd-6-mcp.md` | L6: MCP server (5 tools, stdio transport) |
| `phases/tdd-7-learning.md` | L7: Feedback, suppression, learning |
| `phases/tdd-infra.md` | Server, auth, webhooks, Agent Task API, deployment |
| `phases/phase4b-prompt-specs.md` | LLM prompts (P-EXTRACT, P-TRIAGE, P-VERIFY, P-FIX) |
| `phases/phase4c-ux-specs.md` | PR comment templates, CLI output formats |
| `phases/phase4d-config-spec.md` | .docalign.yml schema and validation |
| `phases/phase5-integration-examples.md` | IE-01 through IE-04 golden examples |
| `phases/phase5-test-strategy.md` | Test tiers, acceptance criteria, coverage targets |

### Architecture (background reference)

| File | Content |
|------|---------|
| `phases/phase3-architecture.md` | System architecture, layer boundaries |
| `phases/phase3-integration-specs.md` | GitHub, LLM, MCP integration details |
| `phases/phase3-error-handling.md` | Error codes, recovery strategies |
| `phases/phase3-infrastructure.md` | Deployment (Railway), CI/CD, monitoring |
| `phases/phase3-security.md` | Threat model, HMAC, path traversal |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BigDanTheOne/docalign](https://github.com/BigDanTheOne/docalign) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
