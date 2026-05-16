---
trigger: always_on
description: APPLY SYMindX architecture standards when working with any file to ensure consistency
---

# SYMindX Workspace Architecture

## Project Overview

SYMindX is an intelligent AI agent framework built with TypeScript, using Bun as the runtime and package manager. The system provides modular, hot-swappable components for memory, emotion, and cognition with multi-platform capabilities.

### Core Architecture Components

**🧠 Core Runtime System** (`mind-agents/`)
- Event-driven architecture with centralized Event Bus
- Hot-swappable module registry for memory, emotion, cognition
- Multi-agent coordination and lifecycle management
- Extension system for platform integrations

**🌐 AI Portal Architecture** (`mind-agents/src/portals/`)
- Multi-provider AI integration (OpenAI, Anthropic, Groq, xAI, Google)
- Vercel AI SDK v5 implementation
- Unified interface for model switching and optimization

**💾 Memory System** (`mind-agents/src/memory/`)
- SQLite, PostgreSQL, Supabase, Neon providers
- Vector embeddings and semantic search
- Conversation persistence and context management

**❤️ Emotion System** (`mind-agents/src/emotion/`)
- 11 distinct emotions (RuneScape-inspired)
- Dynamic emotional state management
- Contextual emotion weighting and responses

**🎯 Cognition Modules** (`mind-agents/src/cognition/`)
- HTN (Hierarchical Task Network) planner
- Reactive response system
- Hybrid cognitive architectures

**🔌 Platform Extensions** (`mind-agents/src/extensions/`)
- Telegram, Slack, Discord integrations
- RuneLite/RuneScape game integration
- Twitter/X social platform support

### Directory Structure

```
symindx/                          # Root project (Bun workspace)
├── .cursor/                      # Cursor IDE configuration
│   ├── rules/                    # Cursor rules (001-017 core)
│   ├── docs/                     # Quick start, architecture, contributing
│   └── tools/                    # Project analyzer, code generator, debugging
├── mind-agents/                  # Core agent runtime system
│   ├── src/
│   │   ├── core/                 # Runtime, EventBus, ModuleRegistry
│   │   ├── portals/              # AI providers (15+ workspaces)
│   │   │   ├── openai/           # OpenAI portal workspace
│   │   │   ├── anthropic/        # Anthropic portal workspace
│   │   │   ├── groq/             # Groq portal workspace
│   │   │   ├── xai/              # xAI portal workspace
│   │   │   ├── google-vertex/    # Google Vertex workspace
│   │   │   ├── google-generative/ # Google Generative workspace
│   │   │   ├── mistral/          # Mistral portal workspace
│   │   │   ├── cohere/           # Cohere portal workspace
│   │   │   ├── azure-openai/     # Azure OpenAI workspace
│   │   │   ├── ollama/           # Ollama portal workspace
│   │   │   ├── lmstudio/         # LM Studio workspace
│   │   │   ├── openrouter/       # OpenRouter workspace
│   │   │   ├── multimodal/       # Multimodal workspace
│   │   │   ├── kluster.ai/       # Kluster.ai workspace
│   │   │   └── vercel/           # Vercel portal workspace
│   │   ├── modules/              # Memory, emotion, cognition modules
│   │   ├── extensions/           # Platform integrations
│   │   ├── characters/           # Character definitions
│   │   ├── utils/                # Shared utilities
│   │   ├── types/                # TypeScript type definitions
│   │   ├── cli/                  # Command-line interface
│   │   └── __tests__/           # Test files
│   ├── data/                     # Runtime data storage
│   ├── scripts/                  # Build and utility scripts
│   ├── docs/                     # Agent documentation
│   └── dist/                     # Build output
├── website/                      # React web interface (Vite + Tailwind)
│   ├── src/
│   │   ├── components/           # React components
│   │   ├── pages/                # Page components
│   │   └── styles/               # Styling files
│   ├── public/                   # Static assets
│   ├── dist/                     # Build output
│   ├── storybook-static/         # Storybook build
│   └── .storybook/              # Storybook configuration
├── docs-site/                    # Documentation site (separate workspace)
├── redirect-package/             # NPM redirect package
├── testing/                      # Comprehensive test suites
├── monitoring/                   # System monitoring tools
├── config/                       # Configuration management
├── .github/                      # GitHub workflows
├── .claude/                      # Claude AI configuration
├── .gitmodules                   # Git submodules
├── docker-compose.yml            # Multi-service deployment
├── Dockerfile                    # Container configuration
└── package.json                  # Root workspace configuration
```

## Development Standards

### Code Quality
- **TypeScript Required**: All code must use TypeScript with strict mode
- **Bun Runtime**: Use Bun for package management and execution
- **Modular Design**: Components must be hot-swappable and independently testable
- **Event-Driven**: Use EventBus for inter-component communication

### Architecture Principles
- **Single Responsibility**: Each module has one clear purpose
- **Dependency Injection**: Use registry pattern for component management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
