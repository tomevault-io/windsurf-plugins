---
trigger: always_on
description: > This file documents the structure and conventions of the Agent Framework
---

# Docs Structure & Design Choices — Agent Framework

> This file documents the structure and conventions of the Agent Framework
> documentation so that agents (AI or human) can maintain it without
> rediscovering decisions.

## Directory layout

```
agent-framework/
├── TOC.yml                    # Single flat table of contents (no nested sub-TOCs)
├── index.yml                  # Landing page (hub page)
├── zone-pivot-groups.yml      # Language pivot definitions
├── docfx.json                 # Build configuration
├── breadcrumb/agent-framework/toc.yml  # Breadcrumb navigation
├── overview/
│   ├── index.md               # "What is Agent Framework" landing
│   └── index.md
├── concepts/                  # Fundamental mental models, semantics, and architecture
│   ├── index.md               # Concepts landing
│   ├── agents/
│   │   ├── index.md           # Agents landing
│   │   ├── conversations/
│   │   └── middleware/
│   ├── workflows/
│   │   ├── index.md           # Workflows landing
│   │   └── advanced/
│   └── harness.md             # Agent Harness composition and architecture
├── get-started/               # 7-step progressive tutorial
│   ├── index.md               # Tutorial landing page
│   ├── your-first-agent.md    # Step 1
│   ├── add-tools.md           # Step 2
│   ├── multi-turn.md          # Step 3
│   ├── memory.md              # Step 4
│   ├── workflows.md           # Step 5
│   ├── harness.md             # Step 6
│   └── hosting.md             # Step 7
├── agents/                    # Generic and built-in agent capability guides
│   ├── index.md               # Agent capabilities landing
│   ├── structured-outputs.md
│   ├── declarative.md
│   ├── observability.md
│   ├── rag.md
│   ├── multimodal.md
│   ├── background-responses.md
│   ├── background-agents.md
│   ├── looping.md
│   ├── planning-and-todos.md
│   ├── security.md
│   ├── tools/                 # 1 page per tool type
│   │   ├── index.md           # Tools overview & landing
│   │   └── ...
├── workflows/                 # Generic and built-in workflow capability guides
│   ├── index.md               # Workflow capabilities landing
│   ├── agents-in-workflows.md
│   ├── human-in-the-loop.md
│   ├── checkpoints.md
│   ├── declarative.md
│   ├── visualization.md
│   ├── observability.md
│   ├── as-agents.md
│   └── orchestrations/       # Multi-agent orchestration patterns
│       ├── index.md           # Orchestrations landing
│       ├── sequential.md
│       ├── concurrent.md
│       ├── handoff.md
│       ├── group-chat.md
│       └── magentic.md
├── integrations/              # Named external things; usually outside services
│   ├── index.md               # Integrations overview & landing
│   ├── by-provider/           # Cross-component provider ecosystem landing pages
│   │   ├── index.md
│   │   ├── microsoft-foundry.md
│   │   ├── microsoft-azure.md
│   │   └── ...
│   └── by-component/          # Canonical implementation guidance by framework surface
│       ├── index.md
│       ├── model-providers/   # Inference providers
│       │   ├── index.md
│       │   └── ...
│       ├── agent-services/    # Managed or protocol-backed remote agent runtimes
│       │   ├── index.md
│       │   ├── a2a.md
│       │   └── ...
│       ├── tools/             # Provider-managed and optional tool integrations
│       │   ├── index.md
│       │   ├── foundry-toolbox.md
│       │   └── shell-tools.md
│       ├── context-providers/ # External before-run/after-run providers
│       │   ├── index.md
│       │   └── ...            # One flat page per external provider
│       ├── middleware/        # External middleware integrations
│       │   └── ...
│       ├── evaluation/        # External evaluation services
│       │   └── ...
│       └── ui/                # Shared UI integrations
│           ├── ag-ui/
│           │   ├── index.md
│           │   └── ...
│           ├── chatkit.md     # Flat page (no subfolder)
│           └── devui/
│               ├── index.md
│               └── ...
├── hosting/                   # Hosting model selection and guides
│   ├── index.md               # Managed vs self-hosted overview
│   ├── azure-functions.md     # Azure Functions and Durable Extension
│   ├── foundry-hosted-agent.md
│   └── self-hosting/
│       ├── index.md           # Shared self-hosting state and protocol choices
│       ├── responses.md
│       ├── openai-endpoints.md
│       ├── telegram.md
│       ├── a2a/
│       │   ├── index.md
│       │   ├── server.md      # Multi-language A2A server guide
│       │   └── dotnet.md
│       └── mcp.md
├── migration-guide/           # SK & AutoGen migration
│   ├── index.md
│   ├── from-autogen/
│   └── from-semantic-kernel/
├── api-docs/                  # API reference (external links)
└── support/                   # FAQ, troubleshooting, upgrade guides
    ├── index.md
    ├── faq.md
    ├── troubleshooting.md
    └── upgrade/
        ├── index.md
        └── ...
```

## Design principles

1. **Progressive then deep**: Get-started (01→07) is a linear tutorial that
   builds complexity step by step. Concepts explain foundational mental models
   and architecture; Agent Capabilities and Workflow Capabilities document

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/azure-ai-docs](https://github.com/MicrosoftDocs/azure-ai-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
