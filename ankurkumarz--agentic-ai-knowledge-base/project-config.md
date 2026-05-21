---
trigger: always_on
description: This file governs how AI agents (Claude Code, Kiro, Gemini CLI, OpenAI Codex, or any other agent) maintain and extend this knowledge base. Read it fully before taking any action.
---

# AGENTS.md — Agentic AI Knowledge Base

This file governs how AI agents (Claude Code, Kiro, Gemini CLI, OpenAI Codex, or any other agent) maintain and extend this knowledge base. Read it fully before taking any action.

---

## What This Repository Is

This is a **persistent, compounding knowledge wiki** about Agentic AI, rendered via MkDocs Material and published to ReadTheDocs. The agent's job is to keep the wiki accurate, well-linked, and organized — not to answer questions in chat. Every insight should land in a file that persists.

The three layers:
- **`raw/`** — immutable source documents you drop in. The agent reads these but never modifies them.
- **`docs/`** — the wiki itself. The agent owns this layer. It creates pages, updates them, and maintains cross-references.
- **`mkdocs.yml`** — the site navigation. The agent updates this when new pages are added.

---

## Repository Layout

```
docs/
├── index.md                        # Home page — executive overview + structure map
├── Introduction/
├── Concepts/                       # Agent definitions, types, foundations
├── AgentHarness/                   # Agent harness concepts and engineering practices
│   ├── agent-harness.md            # What an agent harness is, components, patterns
│   └── harness-engineering.md      # Engineering practices, implementation guidance
├── Architecture/                   # Architecture components, multi-agent, 12-factor
├── DesignPatterns/                 # OpenAI, Gartner patterns
├── AgenticFrameworks/              # LangChain, LangGraph, ADK, CrewAI, etc.
├── AgenticTechStack/               # Tech stack references
├── AgentPlatforms/                 # Vertex AI, AgentCore, Azure AI
├── WorkflowBuilders/               # Workflow engines (OSS, SaaS, etc.)
├── Standards/                      # MCP, A2A, AGENTS.md, OpenSpec, AG-UI
├── ReferenceArchitecture/          # AI Assistant, Automation, RAG, Self-learning
├── ContextEngineering/             # Context challenges, strategies, impl references
├── PromptEngineering/
├── AgentMemory/                    # Memory tiers, LTM strategies, solutions
├── EvaluationFrameworks/           # LLM eval frameworks and platforms
├── Benchmarks/                     # Agent and LLM benchmarks
├── Observability/                  # Goals, solutions, best practices
├── SecurityFrameworks/             # NIST AI RMF, Google SAIF, AWS
├── AgentOps/                       # GenOps, lifecycle management
├── MaturityModels/                 # Gartner, AWS, Google, IDC
├── Marketplace/                    # AWS Marketplace, AgentOps Marketplace
├── RAG/                            # RAG implementation
├── ProductionBestPractices/        # Cross-cutting production guidance
│   ├── README.md                   # Overview and vendor best practices table
│   ├── observability.md
│   ├── state-memory.md
│   ├── deployment.md
│   ├── testing-evaluations.md
│   ├── context-engineering.md
│   ├── security.md
│   └── cost-management.md
├── AllThingsAWS/                   # AWS vendor hub — one-liners + backlinks
├── AllThingsGoogle/                # Google vendor hub — one-liners + backlinks
├── AllThingsMicrosoft/             # Microsoft vendor hub — one-liners + backlinks
├── AllThingsAnthropic/             # Anthropic vendor hub — one-liners + backlinks
└── AllThingsOpenAI/                # OpenAI vendor hub — one-liners + backlinks
                                    # Note: AllThings* pages are index hubs, not content stores.
                                    # Detailed content lives in topical sections; vendor pages link back.

raw/                                # Drop source documents here — agent reads, never modifies
mkdocs.yml                          # Site nav — agent updates when pages are added
```

---

## Section Mapping Guide

When a raw source arrives, use this table to decide which `docs/` directory and which file to update or create.

| Topic in Source | Primary Section | File(s) to Update |
|---|---|---|
| Agent definition, terminology, types | `Concepts/` | `agent-definition.md`, `agent-types.md`, `agent-foundational.md` |
| Agent harness concepts, components, patterns | `AgentHarness/` | `agent-harness.md` |
| Agent harness engineering, implementation, tooling | `AgentHarness/` | `harness-engineering.md` |
| Architecture components, system design | `Architecture/` | `components-selection.md`, `multi-agent-system.md` |
| Design patterns (OpenAI, Gartner, etc.) | `DesignPatterns/` | `openai-patterns.md`, `gartner-patterns.md`, `Readme.md` |
| Development framework (LangChain, LangGraph, CrewAI, etc.) | `AgenticFrameworks/` | `<framework-name>.md` (create if new), `README.md` |
| Cloud platforms (Vertex AI, AgentCore, Azure AI) | `AgentPlatforms/` | `<platform>.md` |
| Workflow engines / builders | `WorkflowBuilders/` | `open-source.md`, `self-hosted.md`, `orchestration.md`, `README.md` |
| Industry standards (MCP, A2A, AGENTS.md) | `Standards/` | `<standard>.md` |
| Reference architectures, blueprints | `ReferenceArchitecture/` | `<architecture-type>.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ankurkumarz/agentic-ai-knowledge-base](https://github.com/ankurkumarz/agentic-ai-knowledge-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
