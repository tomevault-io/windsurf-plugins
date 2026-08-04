---
trigger: always_on
description: > Architecture and development guide for AI assistants contributing to Jarvis AI Platform.
---

# Jarvis AI Platform — Claude Instructions

> Architecture and development guide for AI assistants contributing to Jarvis AI Platform.

---

# Project Context

Jarvis is a **local-first, open-source AI assistant platform** built with **Java 21, Spring Boot 4, Spring AI**, and an **Angular 22 Web UI**.

**GitHub**
https://github.com/sujankim/jarvis-ai-platform

---

# Core Philosophy

Jarvis exists to give users full ownership of their AI.

Core principles:

- Local AI first (Ollama)
- Cloud providers are optional fallback only
- Your data never leaves your machine
- Privacy by architecture, not policy
- Reactive-first backend
- Modular, phase-driven development
- Open-source and developer-focused

---

# Current Phase Status

| Phase | Status | Notes |
|--------|--------|------|
| Phase 1 | ✅ Released | AI Chat + CLI |
| Phase 2 | ✅ Core Complete | Memory + pgvector |
| Phase 3 | ✅ Core Complete | RAG Engine |
| Phase 4 | ✅ Core Complete | Tool Engine + MCP |
| Phase 5 | ✅ Core Complete | Voice Assistant |
| Phase 6 | ✅ Core Complete | Agent System |
| Phase 7 | 🔨 In Progress | Angular 22 Web UI |

---

# AI Architecture Overview

```text
                   User
                     │
         ┌───────────┴───────────┐
         │                       │
       CLI                    REST API
         │                       │
         └───────────┬───────────┘
                     │
                     ▼
              AiOrchestrator
                     │
                     ▼
             PromptAssembler
                     │
     ┌───────────────┼────────────────┐
     │               │                │
Working Memory   Long-Term Memory   RAG Context
     │               │                │
     └───────────────┼────────────────┘
                     │
              Session History
                     │
                     ▼
              ProviderRouter
         ┌───────────┴────────────┐
         │                        │
   OllamaProvider          GeminiProvider
         │                        │
         └───────────┬────────────┘
                     │
              ToolRegistry
                     │
    ┌────────────────┼────────────────┐
    │                │                │
DateTimeTool   CalculatorTool   WeatherTool
                     │
               WebSearchTool
                     │
                MCP Server
                     │
                     ▼
      PostgreSQL + pgvector + Redis

VoiceController
        │
        ▼
VoiceConversationService
        │
        ├── WhisperTranscriptionService
        ├── AiOrchestrator
        └── SystemTextToSpeechService
```

---

# Technology Stack

## Backend (`server/`)

| Layer | Technology |
|--------|------------|
| Language | Java 21 |
| Framework | Spring Boot 4.0.6 |
| AI Framework | Spring AI 2.0.0-M8 |
| Web | Spring WebFlux |
| Security | Spring Security 7 |
| Authentication | JWT + Argon2id |
| Database | PostgreSQL 16 |
| Vector Database | pgvector 0.7.4 |
| Data Access | R2DBC + JDBC |
| Cache | Redis 7 |
| Migrations | Flyway (V1–V18+) |
| Local AI | Ollama |
| Chat Model | llama3.1:8b |
| Embeddings | nomic-embed-text |
| Cloud AI | Google Gemini |
| Tools | Spring AI @Tool + MCP |
| CLI | Spring Shell 4 |
| Mapping | MapStruct 1.6 |
| API Docs | SpringDoc OpenAPI |

---

## Frontend (`client/`)

| Layer | Technology |
|--------|------------|
| Framework | Angular 22 |
| Language | TypeScript 6 |
| UI Components | Angular Material 22 (partial) |
| Styling | Custom SCSS + CSS Variables |
| State | Angular Signals |
| HTTP | Angular HttpClient |
| Routing | Angular Router (lazy loading) |
| Markdown | ngx-markdown |
| Build | Angular CLI 22 + esbuild |

---

# Project Structure

```text
jarvis-ai-platform/

├── server/
│   └── src/main/java/ai/jarvis/
│       ├── ai/
│       │   ├── orchestrator/
│       │   ├── prompt/
│       │   └── provider/
│       │
│       ├── agents/
│       ├── chat/
│       ├── cli/
│       ├── common/
│       ├── config/
│       ├── memory/
│       ├── observability/
│       ├── rag/
│       ├── security/
│       ├── settings/
│       ├── tools/
│       │   ├── builtin/
│       │   └── mcp/
│       ├── user/
│       └── voice/
│
└── client/
    └── src/app/
        ├── core/
        │   ├── guards/
        │   ├── interceptors/
        │   ├── models/
        │   └── services/
        │
        ├── shared/
        │   └── components/
        │
        └── features/
            ├── agents/
            ├── chat/
            ├── documents/
            ├── login/
            ├── memory/
            ├── settings/
            └── voice/
```

---

# Backend Architecture Rules

## 1. AiProvider Interface Is Sacred

Every AI provider implements `AiProvider`.

Rules:

- Provider selection only through `ProviderRouter`
- Providers receive `ToolRegistry`
- Never inject provider implementations directly
- Never bypass ProviderRouter

---

## 2. Dependency Direction (STRICT)

```text
CLI
 ↓
Controllers
 ↓
Services
 ↓
Providers
 ↓
Database
```

Never bypass layers.

---

## 3. AiOrchestrator Is The Only AI Coordinator

Responsibilities:

- Load session history
- Load working memory
- Load long-term memory
- Load RAG context
- Assemble prompt
- Select provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sujankim/jarvis-ai-platform](https://github.com/sujankim/jarvis-ai-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
