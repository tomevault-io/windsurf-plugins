---
trigger: always_on
description: **Unified tools, agents, and resources monorepo — agent services, MCP servers, utilities, reference libraries, and security wordlists.**
---

# gbs-tools-and-resources

**Unified tools, agents, and resources monorepo — agent services, MCP servers, utilities, reference libraries, and security wordlists.**

---

## ⚠️ CRITICAL GUARDRAILS (READ FIRST)

1. **NEVER push without permission** — Even small fixes require express user permission. No exceptions.
2. **NEVER expose secrets** — No API keys, tokens, credentials in git, logs, or conversation.
3. **NEVER force push or skip tests** — 100% passing tests required.
4. **ALWAYS read parent CLAUDE.md** — `~/CLAUDE.md` for org-wide standards.
5. **ALWAYS use Definition of Ready** — 100% clear requirements before implementation.

---

## Project Overview

This monorepo consolidates 17 Glass Box packages into a single discoverable location. Each package retains its own structure, language, and build tooling — there is no unified build system at the monorepo root.

**Initial consolidation:** 2026-03-01 (7 utility/research repos)
**Agent consolidation:** 2026-03-08 (4 agent packages added)
**Pattern follows:** `internal-tools` (absorbed command-center, glass-box-board, glass-box-hub, Squeegee on 2026-03-01)

---

## Packages

### Agent Services

| Package | Stack | Purpose | Deploy |
|---------|-------|---------|--------|
| [`packages/spectacles/`](packages/spectacles/) | Python 3.12, FastAPI, Playwright, Gemini | Browser automation + documentation intelligence curator | Cloud Run: `glassbox-spectacles` |
| [`packages/merus-expert/`](packages/merus-expert/) | Python 3.12, FastAPI, Claude, Gemini | MerusCase domain agent — 13 tools, SSE streaming | Cloud Run: `merus-expert` |
| [`packages/agent-swarm/`](packages/agent-swarm/) | NestJS 11, TypeScript, Socket.io | DAG-based multi-agent task orchestration | Standalone NestJS library (canonical) |
| [`packages/agentic-debugger/`](packages/agentic-debugger/) | Node.js, Claude Code, GitHub Actions | Automated CI test failure debugging agent | Standalone CI debugging agent (canonical) |

### Operations & Audit

| Package | Stack | Purpose | Deploy |
|---------|-------|---------|--------|
| [`packages/compliance-auditor/`](packages/compliance-auditor/) | Node.js 20, Fastify 5, Prisma, Zod | SOC2 + HIPAA compliance code scanning | Cloud Run |
| [`packages/gbs-integration-validator/`](packages/gbs-integration-validator/) | Node.js 20, Fastify 5, Zod | API integration validation (7 platforms) | Cloud Run |
| [`packages/invoice-reconciliation-tester/`](packages/invoice-reconciliation-tester/) | Node.js 20, Fastify 5, Prisma, Faker | Invoice reconciliation algorithm testing | Cloud Run |

### Utilities & Libraries

| Package | Stack | Purpose |
|---------|-------|---------|
| [`packages/hindsight/`](packages/hindsight/) | Python/FastAPI, Next.js, pgvector, Rust | Agent memory system (retain/recall/reflect) |
| [`packages/mcp-servers/`](packages/mcp-servers/) | Python, Node.js, MCP SDK | MCP server collection (kb-api, kb-db, wc-paralegal, social-media) |
| [`packages/phileas/`](packages/phileas/) | Java 11+, Maven | PII/PHI redaction library (30+ entity types) |
| [`packages/yevrah_terminal/`](packages/yevrah_terminal/) | Python 3.x, Groq, Cohere | Terminal legal research (keyword + semantic search) |
| [`packages/merus-test-data-generator/`](packages/merus-test-data-generator/) | Python 3.12, reportlab, Faker | WC test case generator (10,000+ templated PDFs across 97+ subtypes; AMA Guides 5th Ed. impairment content, 30 edge case scenarios, Browserless MerusCase batch integration) |

### Reference

| Package | Purpose |
|---------|---------|
| [`packages/awesome-agent-skills/`](packages/awesome-agent-skills/) | Curated catalog of 180+ AI agent skills |
| [`packages/awesome-claude-code/`](packages/awesome-claude-code/) | Curated list of skills, hooks, slash-commands, agents, and plugins for Claude Code |
| [`packages/cli-anything/`](packages/cli-anything/) | Claude Code plugin — transform GUI apps into agent-native CLIs (GIMP, Blender, etc.) |
| [`packages/ui-ux-pro-max-skill/`](packages/ui-ux-pro-max-skill/) | AI design intelligence skill for professional UI/UX across platforms |
| [`packages/SecLists-GBS-Branch/`](packages/SecLists-GBS-Branch/) | Security testing wordlists (placeholder) |

---

## Commands

Each package builds independently. See each package's own README.md or CLAUDE.md for build/run/test commands.

### Quick navigation

```bash
# Agent services
cd packages/spectacles/
cd packages/merus-expert/
cd packages/agent-swarm/
cd packages/agentic-debugger/

# Utilities
cd packages/hindsight/
cd packages/mcp-servers/
cd packages/phileas/
cd packages/yevrah_terminal/
cd packages/merus-test-data-generator/
```

### Package-specific tests

```bash
# spectacles (Python)
cd packages/spectacles && pytest

# merus-expert (Python)
cd packages/merus-expert && pytest

# hindsight (Python + Next.js)
cd packages/hindsight && pytest

# yevrah_terminal (Python)
cd packages/yevrah_terminal && pytest

# phileas (Java/Maven)
cd packages/phileas && mvn test

# mcp-servers (varies by server)
cd packages/mcp-servers/servers/[server-name] && npm test
```

---

## Architecture

```
gbs-tools-and-resources/
├── CLAUDE.md                         # This file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Glass-Box-Solutions-Inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
