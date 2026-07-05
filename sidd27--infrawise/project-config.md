---
trigger: always_on
description: Validates the full adapter stack against real AWS services emulated locally. No AWS account needed.
---

# Infrawise — AI instructions

## Project goal

**Genesis pain:** Developers had to copy-paste DynamoDB schemas, RDS table definitions, and Lambda trigger shapes from the AWS console into their AI coding assistant every single session. Manual, repetitive, breaks flow, and still produces wrong code because the pasted context is incomplete.

**What Infrawise is:** The canonical tool for infra-aware AI coding. When a developer thinks "how does my AI assistant know my DynamoDB schema without me pasting it?" — the answer should be Infrawise. The positioning goal is to own the "infra-aware AI coding" category the way Prettier owns formatting or Husky owns git hooks.

**Two product pillars:**

*Context* (developer experience) — AI coding assistants know your live infrastructure. Exact partition keys, sort keys, GSIs, Lambda trigger event shapes, queue DLQ coverage, API routes and their Lambda bindings. The developer never looks at the AWS console mid-coding again. Write correct code the first time because the AI already knows the schema.

*Guard* (mistake prevention) — Surface costly infra mistakes before they ship. Wrong SQS visibility timeout causing duplicate Lambda processing. Missing DLQ silently dropping failed messages. A Lambda scanning a DynamoDB table without a GSI. RDS queries on unindexed columns. These are expensive to discover in production. Infrawise warns at coding time, not incident time.

**What it covers today:**
- AWS: DynamoDB, Lambda, SQS, SNS, SSM Parameter Store, Secrets Manager, EventBridge, RDS, API Gateway, S3, CloudWatch Logs, Cognito, Kinesis, MSK (clusters), ElastiCache, CloudWatch metrics (opt-in runtime signals)
- Databases: PostgreSQL, MySQL, MongoDB
- Messaging: Apache Kafka via `kafkajs` — broker-agnostic (self-hosted, Confluent, Redpanda, or Amazon MSK). Producer/consumer-to-topic mapping is extracted from application code (AST scan, always-on, no config key) and surfaced as topic nodes via `get_topic_details`. Distinct from the Amazon MSK *Lambda trigger* (detected from the event-source ARN, with event shape `event.records[topic][0].value`).
- IaC: Terraform, CDK, CloudFormation (local file parsing for drift detection, plus stack outputs / cross-stack exports)

**How it works:** `infrawise analyze` extracts infrastructure into an in-memory graph, runs rule-based analyzers to generate findings, then either prints a report (CLI) or serves 20 MCP tools (server mode) that AI assistants call to get precise context before writing code.

**Strategic bets:**
- MCP is the primary integration surface (Claude Code, Cursor, any MCP-capable editor). `infrawise check` is the standalone CI/CD gate — runs a fresh analysis and exits non-zero when findings reach `--fail-on` severity (default high), reaching teams not yet using AI editors.
- TypeScript/Node is the current runtime; language is a limitation to overcome, not a design choice. The cloud extraction layer is already language-agnostic. AST scanning can expand to Python, Go, etc. over time.
- Zero-config fast path is the unlock for adoption. The "aha moment" must happen in under 2 minutes from install — `npx infrawise start` auto-discovers AWS credentials and infra, no infrawise.yaml required. `start` is the entry point; do not add new setup commands (`init`, `doctor`, etc.) that front-load friction.
- The command surface is deliberately five verbs, one per user need: `start` (onboard), `analyze` (full report), `check` (CI gate), `serve` (MCP server — `--stdio` for editors, HTTP by default), `doctor` (diagnostic escape hatch). `stdio` is a hidden backcompat alias for `serve --stdio` (older `.mcp.json` files invoke it). The interactive wizard lives in `src/cli/interactive-setup.ts` (`runInit`), reachable only via `start --interactive`; it is not its own command. Do not re-add `init`, `auth`, or `dev` as commands — their jobs are subsumed by `start`/`serve`.

**Who uses it:** Solo project. One developer (Sidd) is the only contributor. No team conventions apply.

## Pre-release checklist — NO EXCEPTIONS

Before running `pnpm release <patch|minor|major>`, every item below must be current. Check each one — do not skip.

**Files to verify are in sync:**

| File | What to check |
|---|---|
| `README.md` | CLI reference table, MCP tools table, Analysis capabilities table, Configuration section |
| `AGENTS.md` | MCP tool reference section — all tools, inputs, return shape, when to call |
| `llms.txt` | Quick start commands, MCP tools list (count + names match `src/server/index.ts`) |
| `src/server/index.ts` | Tool descriptions — purpose + when to call + when NOT to call (TDQS criteria) |
| `website/src/pages/index.astro` | `softwareVersion` in `SoftwareApplication` JSON-LD schema (hardcoded string, search for `"softwareVersion"`) |

**Auto-updated by `pnpm release` — no action needed:**
- `package.json` — version
- `server.json` — version (MCP Registry manifest)
- `docs/architecture.svg` — regenerated from `docs/architecture.yml` before commit
- Git commit, tag, push, draft GitHub release

**If you change the architecture diagram:**
1. Edit `docs/architecture.yml` — the single source of truth for all diagram variants
2. Run `pnpm build-arch` — generates two outputs:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sidd27/infrawise](https://github.com/Sidd27/infrawise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
