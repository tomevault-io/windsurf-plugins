---
trigger: always_on
description: This file is read automatically by OpenClaw, Claude Code, and other AI coding
---

# n8n-to-claw — Agent Context

This file is read automatically by OpenClaw, Claude Code, and other AI coding
agents to understand the codebase before working on it.

## What this project does

`n8n-to-claw` is a CLI tool that converts [n8n](https://n8n.io) workflow JSON
into [OpenClaw](https://openclaw.ai)-compatible skills (`SKILL.md` + `skill.ts`).
Transpilation is usually an LLM call; **deterministic templates** handle some
deterministic HTTP templates (linear or IF + GET chain; webhook allowed; see
`src/transpile/deterministic/linear-http-chain.ts`).

## Architecture — three-stage pipeline

```
Input (file or n8n API)
       │
       ▼
┌─────────────┐     ┌────────────────────────────┐     ┌───────────┐
│  Parse      │────▶│  Transpile (template|LLM)  │────▶│  Package  │
│  IR types   │     │  validate skill.ts (tsc) │     │  write    │
└─────────────┘     └────────────────────────────┘     └───────────┘
```

## Source layout

```
src/
  ir/
    types.ts               — WorkflowIR and all IR interfaces (the central contract)
  parse/
    n8n-schema.ts          — Raw n8n JSON types (pre-normalization)
    categorize.ts          — Node type → category mapping table + trigger detection
    categorize.test.ts
    parser.ts              — parse() function, ParseError
    parser.test.ts
    quality.ts             — IR readiness score (IRQuality) from parse warnings
  adapters/
    file.ts                — Load raw JSON from a local file
    api.ts                 — Load raw JSON from the n8n REST API
  transpile/
    llm.ts                 — OpenAI-compatible LLM client, timeout, 429/5xx retry
    prompt.ts              — Build LLM messages from WorkflowIR (includes few-shot example)
    output-parser.ts       — Extract SKILL.md + skill.ts from LLM response
    validate.ts            — Run tsc --noEmit on generated skill.ts
    deterministic/
      linear-http-chain.ts — Deterministic templates: linear / IF + HTTP GET (no LLM)
    transpile.ts           — Template first, else LLM → validate → retry → draft fallback
    *.test.ts
  utils/
    logger.ts              — DEBUG=n8n-to-claw structured logging; zero cost when disabled
  package/
    package.ts             — Write SKILL.md, skill.ts, warnings.json, skill-meta.json, creds example
    package.test.ts
  coverage/
    node-coverage.ts       — Scan test-fixtures → mapping stats; `npm run coverage:nodes`
  cli/
    index.ts               — CLI entry point (parseArgs, orchestrates all stages)
    debug-bundle.ts        — Writes reproducible transpile debug artifacts per run
  integration.test.ts      — Full pipeline tests with mocked LLM

skills/
  n8n-to-claw/
    SKILL.md               — OpenClaw skill so agents can invoke this tool

test-fixtures/
  notify-slack-on-postgres.json   — Schedule → Postgres → IF → Slack
  github-webhook-to-slack.json    — Webhook → IF branch → Slack API
  ai-support-chatbot.json         — LangChain AI agent with ai_* connections
  daily-hacker-news-digest.json   — Cron → HTTP → Code → Email
  sync-crm-with-custom-nodes.json — Community node, stickyNote, Google Sheets
  schedule-http-ping.json         — Deterministic: schedule → HTTP GET chain
  webhook-http-ping.json          — Deterministic: webhook → HTTP GET chain
  schedule-noop-http-ping.json    — Deterministic: schedule → noOp → HTTP GET
  webhook-if-http-ping.json       — Deterministic: webhook → IF ($json field) → HTTP / noOp

examples/
  github-pr-review-notifier/      — Sample SKILL.md + skill.ts output
  daily-hacker-news-digest/       — Sample SKILL.md + skill.ts output

Dockerfile                 — Multi-stage image: CLI dist + web UI (Express + static Vite build)
docker-compose.yml         — `docker compose up` for local / cloud deployment
.dockerignore

web/
  server.ts                — Express API server (parse + transpile routes)
  vite.config.ts           — Vite config for React SPA
  src/
    App.tsx                — Main app with step-by-step flow
    components/            — UploadPanel, ParseResults, TranspileForm, OutputViewer
    hooks/useWorkflow.ts   — State management hook
    api.ts                 — Fetch wrappers for /api/parse, /api/transpile

docs/
  architecture.md          — Deeper architecture notes
  ir-schema.md             — WorkflowIR field-by-field reference

scripts/
  setup.sh                 — Install deps and verify environment
```

## Key invariants — do not break these

1. **`src/ir/types.ts` is the single source of truth** for the IR shape. Parse
   produces it, transpile consumes it. Changes here ripple everywhere.

2. **`src/parse/n8n-schema.ts` and `src/ir/types.ts` must stay separate.**
   The schema types represent untrusted raw input; IR types represent normalized,
   validated data. Never merge them.

3. **The `raw` field on `WorkflowIR` and `IRNode` is a reference, not a clone.**
   Do not mutate it in the transpile or package stages.

4. **The parse stage never calls the LLM.** The transpile stage never reads
   from disk. The package stage never calls the LLM or the n8n API. Stages
   are strictly separated. The deterministic template path does not call the LLM.

5. **`tsc --noEmit` must pass at all times.** The tsconfig uses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [just-claw-it/n8n-to-claw](https://github.com/just-claw-it/n8n-to-claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
