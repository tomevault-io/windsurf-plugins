---
trigger: always_on
description: Agent Skills is a repository of skills, prompts, and MCP configurations for AI coding agents working with Azure SDKs and Microsoft AI Foundry services.
---

# Copilot Instructions for Agent Skills

## Project Overview

Agent Skills is a repository of skills, prompts, and MCP configurations for AI coding agents working with Azure SDKs and Microsoft AI Foundry services.

## ⚠️ Fresh Information First

**Azure SDKs and Foundry APIs change constantly. Never work with stale knowledge.**

Before implementing anything with Azure/Foundry SDKs:

1. **Search official docs first** — Use the Microsoft Docs MCP (`microsoft-docs`) to get current API signatures, parameters, and patterns
2. **Verify SDK versions** — Check `pip show <package>` for installed versions; APIs differ between versions
3. **Don't trust cached knowledge** — Your training data is outdated. The SDK you "know" may have breaking changes.

**If you skip this step and use outdated patterns, you will produce broken code.**

---

## Core Principles

Apply these principles to every task.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- If you write 200 lines and it could be 50, rewrite it.

**The test:** Would a senior engineer say this is overcomplicated? If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

**The test:** Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution (TDD)

**Define success criteria. Loop until verified.**

| Instead of... | Transform to... |
|---------------|-----------------|
| "Add validation" | "Write tests for invalid inputs, then make them pass" |
| "Fix the bug" | "Write a test that reproduces it, then make it pass" |
| "Refactor X" | "Ensure tests pass before and after" |

---

## Repository Structure

```
AGENTS.md                # Agent configuration template

.github/
├── skills/              # Backward-compat symlinks to plugin skills
│   └── */SKILL.md       # Each skill has YAML frontmatter + markdown body
├── plugins/             # Language-based plugin bundles (azure-sdk-python, etc.)
│   └── azure-sdk-*/     # Each bundle has skills/, commands/, agents/
├── prompts/             # Reusable prompt templates
├── agents/              # Agent persona definitions (backend, frontend, infrastructure, planner, presenter)
├── scripts/             # Automation scripts (doc scraping)
├── workflows/           # GitHub Actions (daily doc updates)
└── copilot-instructions.md

docs/                    # Generated llms.txt files (daily workflow) - GitHub Pages hosted
├── llms.txt             # Links + summaries
└── llms-full.txt        # Full content

skills/                  # Symlinks for backward compatibility
├── python/              # -> ../.github/skills/*-py
├── dotnet/              # -> ../.github/skills/*-dotnet
├── typescript/          # -> ../.github/skills/*-ts
├── java/                # -> ../.github/skills/*-java
└── rust/                # -> ../.github/skills/*-rust

.vscode/
└── mcp.json             # MCP server configurations
```

## Skills

Skills are domain-specific knowledge packages in `.github/skills/`. Each skill has a `SKILL.md` with:
- **YAML frontmatter** (`name`, `description`) — triggers skill loading
- **Markdown body** — loaded only when skill activates

### Skill Naming Convention

Skills use language suffixes for discoverability:

| Language | Suffix | Examples |
|----------|--------|----------|
| **Core** | — | `mcp-builder`, `skill-creator`, `copilot-sdk` |
| **Python** | `-py` | `azure-ai-inference-py`, `azure-cosmos-db-py`, `azure-ai-projects-py` |
| **.NET** | `-dotnet` | `azure-ai-inference-dotnet`, `azure-resource-manager-cosmosdb-dotnet` |
| **TypeScript** | `-ts` | `azure-ai-inference-ts`, `azure-ai-agents-ts`, `frontend-ui-dark-ts` |
| **Java** | `-java` | `azure-ai-inference-java`, `azure-cosmos-java` |
| **Rust** | `-rust` | `azure-identity-rust`, `azure-cosmos-rust` |

### Featured Skills

| Skill | Purpose |
|-------|---------|
| `azure-search-documents-py` | Search SDK patterns, vector/hybrid search, agentic retrieval |
| `azure-ai-agents-py` | Low-level agents SDK for CRUD, threads, streaming, tools |
| `azure-ai-voicelive-py` | Real-time voice AI with Azure AI Voice Live SDK |
| `azure-ai-projects-py` | High-level Foundry project client, versioned agents, evals |
| `frontend-ui-dark-ts` | Dark theme UI patterns (Vite + React + Tailwind + Framer Motion) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/skills](https://github.com/microsoft/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
