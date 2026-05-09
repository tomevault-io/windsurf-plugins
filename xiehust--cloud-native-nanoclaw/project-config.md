---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClawBot Cloud — a multi-tenant AI assistant platform on AWS. Users create Bots via a web console, connect messaging channels (Telegram, Discord, Slack, Feishu/Lark), and Bots run Claude Agents in AgentCore microVMs with independent memory, conversations, and scheduled tasks.

## Commands

```bash
# Install all dependencies (from repo root)
npm install

# Build all packages
npm run build --workspaces

# Build a single package
npm run build -w shared
npm run build -w control-plane
npm run build -w agent-runtime
npm run build -w web-console
npm run build -w infra

# Type-check without emitting
npm run typecheck -w control-plane
npm run typecheck -w agent-runtime
npm run typecheck -w infra
npm run typecheck -w shared

# Run tests (control-plane only — vitest)
npm test -w control-plane
npm run test:watch -w control-plane   # watch mode

# Local development
npm run dev -w control-plane          # tsx watch, port 3000
npm run dev -w web-console            # vite, port 5173

# CDK infrastructure
cd infra
npx cdk synth                         # synthesize CloudFormation
npx cdk deploy --all                  # deploy all stacks
npx cdk bootstrap                     # one-time per account/region
```

**Build order matters:** `shared` must be built before packages that depend on it (`control-plane`, `agent-runtime`).

## Architecture

NPM workspaces monorepo with 5 packages. ESM throughout (`"type": "module"`), TypeScript strict mode, target ES2022.

### Package dependency graph

```
shared ◄── control-plane
       ◄── agent-runtime

infra (standalone — references no other packages)
web-console (standalone — talks to control-plane via REST)
```

### Package roles

- **shared** (`@clawbot/shared`) — Domain types (User, Bot, Channel, Message, Task, Session), Channel Adapter interfaces, XML formatter for agent context, text utilities. Exports via subpath exports: `@clawbot/shared/types`, `@clawbot/shared/channel-adapter`, `@clawbot/shared/xml-formatter`, `@clawbot/shared/text-utils`.
- **control-plane** (`@clawbot/control-plane`) — Fastify HTTP server on ECS Fargate. Handles webhook ingestion (Telegram/Slack), Discord Gateway (discord.js with leader election), Feishu Gateway (Lark SDK WSClient with leader election), REST API for the web console (JWT-authed via Cognito, including admin APIs), SQS FIFO message dispatching to AgentCore, SQS reply consumption via Channel Adapter Registry, channel health checking, and native CLAUDE.md memory management (bot-level + group-level).
- **agent-runtime** (`@clawbot/agent-runtime`) — Runs inside AgentCore microVMs. Wraps Claude Agent SDK with MCP tools (send_message, schedule_task, etc.). Manages S3 session sync, native CLAUDE.md memory (via Claude Code settingSources), STS ABAC scoped credentials, and per-bot tool/skill whitelist enforcement (PreToolUse hook). Exposes `/invocations` and `/ping` endpoints.
- **infra** (`@clawbot/infra`) — AWS CDK (TypeScript). 6 stacks: Foundation (VPC, S3, DynamoDB, SQS, ECR), Auth (Cognito), Agent (IAM ABAC roles), ControlPlane (ALB, ECS, WAF), Frontend (CloudFront + S3), Monitoring (CloudWatch).
- **web-console** (`@clawbot/web-console`) — React 19 SPA with Vite, TailwindCSS, AWS Amplify for Cognito auth. Pages: Login, Dashboard, BotDetail (tabs: Overview/Channels/Conversations/Tasks/Memory/Files/Tools/Settings), ChannelSetup, Messages, Tasks, MemoryEditor (3 tabs: Shared/BotMemory/GroupMemory), Settings (Anthropic API provider config), Admin UserList/UserDetail.

### Message flow

User message → Channel webhook/Gateway → Control Plane (signature verification, DynamoDB store) → SQS FIFO → SQS consumer (quota check, concurrency control) → AgentCore invocation → Claude Agent SDK `query()` (preset append mode, native CLAUDE.md) → MCP tools → response stored in DynamoDB → Channel Adapter Registry → Channel API reply.

Agent intermediate messages: MCP `send_message` → SQS Standard reply queue → Reply Consumer → Channel Adapter → Channel API.

SQS FIFO provides per-group message ordering with cross-group parallelism. Discord and Feishu use Gateway (WebSocket) with DynamoDB-based leader election instead of webhooks.

### Security model

- Cognito JWT on all `/api/*` routes
- Per-channel webhook signature verification (Telegram secret token, Discord Ed25519, Slack HMAC-SHA256)
- ABAC via STS SessionTags — agents can only access their owner's S3 paths and DynamoDB records
- Channel tokens and Anthropic API keys in Secrets Manager, never exposed to agents
- Fargate in private subnets, WAF rate limiting

### Data layer

- **DynamoDB** — 7 tables for Users, Bots, Channels, Messages, Tasks, Sessions, Groups
- **S3** — Session state and CLAUDE.md memory files
- **Secrets Manager** — Channel API tokens (Telegram, Discord, Slack, Feishu), per-user Anthropic API keys
- **EventBridge Scheduler** — Scheduled tasks → SQS → Agent

## Key Libraries

| Library | Version | Used in |
|---------|---------|---------|
| Fastify | 5.2 | control-plane, agent-runtime |
| AWS SDK v3 | 3.700+ | control-plane, agent-runtime |
| Claude Agent SDK | 0.2.76 | agent-runtime |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiehust/cloud-native-nanoclaw](https://github.com/xiehust/cloud-native-nanoclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
