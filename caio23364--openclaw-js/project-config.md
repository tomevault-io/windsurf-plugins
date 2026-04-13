---
trigger: always_on
description: **Welcome!** If you are an AI Coding Agent tasked to modify, review, or debug this project, read this document first. It contains essential architectural constraints, coding conventions, and project-specific knowledge.
---

# 🤖 OpenClaw JS — Manual for AI Agents

**Welcome!** If you are an AI Coding Agent tasked to modify, review, or debug this project, read this document first. It contains essential architectural constraints, coding conventions, and project-specific knowledge.

---

## 1. Project Overview

**OpenClaw JS** (v2026.2.14) is a security-hardened, multi-channel personal AI assistant written in TypeScript. It provides a unified interface to interact with multiple AI providers (Anthropic Claude, OpenAI GPT, Google Gemini, DeepSeek, etc.) across various messaging channels (WhatsApp, Telegram, Discord, Slack, Signal, Matrix, WebChat).

### Key Goals
- **Security-first**: Defense-in-depth with origin validation, rate limiting, SSRF protection, sandboxing
- **Multi-provider AI**: Support 15+ AI vendors with unified interface
- **Multi-channel**: Unified inbox across 7+ messaging platforms
- **Local-first**: Gateway binds to localhost by default, data stays local
- **Extensible**: Skills platform for custom tools and integrations

---

## 2. Technology Stack

| Layer | Technology |
|-------|------------|
| **Runtime** | Node.js >= 22.0.0 (ES2022, ES Modules) |
| **Language** | TypeScript 5.7+ with strict mode |
| **Build** | `tsc` (TypeScript compiler) |
| **Testing** | Vitest (48 tests, all passing) |
| **Linting** | ESLint 9.x with TypeScript plugin |
| **Formatting** | Prettier |
| **Logging** | Pino (structured logging) |
| **Gateway** | Express + WebSocket (`ws`) + Socket.IO |

### Dependencies Overview
- **23 production dependencies** (reduced from 50)
- **11 dev dependencies** (reduced from 17)
- **55% dependency reduction** achieved by using native Node.js APIs

---

## 3. Project Structure

```
openclaw-js/
├── src/
│   ├── index.ts              # Main entry point, OpenClaw class
│   ├── agents/               # Agent runtime, session management, tools
│   │   └── index.ts          # AgentRuntime class, built-in tools
│   ├── browser/              # Puppeteer browser automation
│   ├── channels/             # Messaging channel integrations (lazy-loaded)
│   │   ├── index.ts          # ChannelManager
│   │   ├── whatsapp.ts       # @whiskeysockets/baileys
│   │   ├── telegram.ts       # grammy
│   │   ├── discord.ts        # discord.js
│   │   ├── slack.ts          # @slack/bolt
│   │   ├── signal.ts         # signal-cli (child_process)
│   │   ├── matrix.ts         # matrix-js-sdk
│   │   └── webchat.ts        # Socket.IO
│   ├── cli/                  # Command-line interface
│   │   └── index.ts          # Commander-based CLI with all commands
│   ├── cron/                 # Scheduled tasks (node-cron)
│   ├── gateway/              # WebSocket/HTTP gateway
│   │   ├── index.ts          # Gateway class, JSON-RPC protocol
│   │   └── commands.ts       # Chat command parsing (/status, /think, etc.)
│   ├── heartbeat/            # Periodic health checks
│   ├── identity/             # Device identity management
│   ├── memory/               # Agent memory system (sqlite/markdown)
│   ├── metrics/              # Metrics collection and dashboard
│   ├── providers/            # AI provider integrations (lazy-loaded)
│   │   ├── index.ts          # ProviderManager
│   │   ├── vendors.ts        # Vendor registry (15+ providers)
│   │   ├── anthropic.ts      # Anthropic SDK
│   │   ├── openai.ts         # OpenAI SDK (also for OpenAI-compatible)
│   │   ├── google.ts         # Google Generative AI
│   │   ├── failover.ts       # Provider failover logic
│   │   ├── streaming.ts      # Streaming response handling
│   │   └── media.ts          # Media processing for multimodal
│   ├── runtime/              # Docker runtime support
│   ├── security/             # Security controls (CRITICAL)
│   │   ├── index.ts          # RateLimiter, OriginValidator, AuditLogger, InputValidator
│   │   └── sandbox.ts        # Command sandbox, path traversal protection
│   ├── service/              # System service management (systemd/OpenRC)
│   ├── skills/               # Skills platform with audit
│   ├── tunnel/               # Tunnel providers (Cloudflare, ngrok, Tailscale)
│   ├── types/                # TypeScript type definitions
│   │   └── index.ts          # All core types (Session, Message, Agent, etc.)
│   └── utils/                # Utilities
│       ├── config.ts         # Configuration management (~/.openclaw/)
│       ├── logger.ts         # Pino logger setup
│       ├── helpers.ts        # ID generation, StateStore
│       └── index.ts          # Utility exports
├── test/                     # Vitest test suites
│   ├── security.test.ts      # Security module tests (48 tests)
│   ├── agents.test.ts        # Agent runtime tests
│   ├── providers.test.ts     # Provider tests
│   └── setup.ts              # Test setup (auto mock restore)
├── bin/                      # CLI entry point
├── Dockerfile                # Multi-stage Docker build
├── docker-compose.yml        # Docker Compose configuration
├── docker-build.sh           # Docker build script
├── DOCKER.md                 # Docker documentation
├── .dockerignore             # Docker ignore rules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Caio23364) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
