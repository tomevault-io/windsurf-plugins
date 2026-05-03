---
trigger: always_on
description: This repository provides comprehensive security rules for Claude Code, covering web applications, AI/ML systems, and agentic AI.
---

# CLAUDE.md - Secure Coding Rules for Claude Code

This repository provides comprehensive security rules for Claude Code, covering web applications, AI/ML systems, and agentic AI.

## Project Overview

**Purpose**: Open-source security rules that guide Claude Code to generate secure code by default

**Coverage**:
- OWASP Top 10 2025 (web application security)
- OWASP MCP Top 10 2025 (Model Context Protocol security)
- AI/ML security (NIST AI RMF, MITRE ATLAS, Google SAIF)
- Agentic AI security (tool use, autonomy, sandboxing)
- Language-specific rules (Python, JavaScript, TypeScript, Go, Rust, Java, C#, Ruby, R, C++, Julia, SQL)
- Backend frameworks (FastAPI, Express, Django, Flask, NestJS)
- AI/ML frameworks (LangChain, CrewAI, AutoGen, Transformers, vLLM, Triton, TorchServe, Ray Serve, BentoML, MLflow, Modal)
- Frontend frameworks (React, Next.js, Vue, Angular, Svelte)

## Repository Structure

```
claude-secure-coding-rules/
├── rules/
│   ├── _core/                      # Foundation rules (apply to all projects)
│   │   ├── owasp-2025.md          # OWASP Top 10 2025 security rules
│   │   ├── mcp-security.md        # Model Context Protocol (MCP) security rules
│   │   ├── ai-security.md         # AI/ML system security rules
│   │   └── agent-security.md      # Agentic AI security rules
│   │
│   ├── languages/                  # Language-specific security rules
│   │   ├── python/CLAUDE.md       # Deserialization, subprocess, path traversal, crypto, SQL
│   │   ├── javascript/CLAUDE.md   # eval, prototype pollution, DOM security, Node.js
│   │   ├── typescript/CLAUDE.md   # Type safety, validation, any types
│   │   ├── go/CLAUDE.md           # Concurrency, context, templates, error handling
│   │   ├── rust/CLAUDE.md         # unsafe blocks, FFI, memory safety, crypto
│   │   ├── java/CLAUDE.md         # Serialization, JNDI, reflection, streams
│   │   ├── csharp/CLAUDE.md       # .NET patterns, LINQ injection, assemblies
│   │   ├── ruby/CLAUDE.md         # Metaprogramming, ERB, mass assignment
│   │   ├── r/CLAUDE.md            # Shiny apps, data security, package verification
│   │   ├── cpp/CLAUDE.md          # Memory safety, buffer overflows, smart pointers
│   │   ├── julia/CLAUDE.md        # Metaprogramming, type safety, serialization
│   │   └── sql/CLAUDE.md          # Injection, permissions, stored procedures
│   │
│   ├── backend/                    # Backend framework rules
│   │   ├── fastapi/CLAUDE.md      # Pydantic validation, JWT, authorization, CORS, AI APIs
│   │   ├── express/CLAUDE.md      # Helmet, sessions, rate limiting, file uploads
│   │   ├── django/CLAUDE.md       # ORM, CSRF, templates, settings
│   │   ├── flask/CLAUDE.md        # Werkzeug, sessions, blueprints, extensions
│   │   ├── nestjs/CLAUDE.md       # Decorators, guards, pipes, interceptors
│   │   ├── langchain/CLAUDE.md    # Prompt injection, tool security, chains, RAG
│   │   ├── crewai/CLAUDE.md       # Multi-agent trust, delegation, memory isolation
│   │   ├── autogen/CLAUDE.md      # Code execution, human-in-loop, sandboxing
│   │   ├── transformers/CLAUDE.md # Model loading, tokenizers, fine-tuning
│   │   ├── vllm/CLAUDE.md         # KV cache, PagedAttention, batching security
│   │   ├── triton/CLAUDE.md       # GPU isolation, ensemble security, gRPC
│   │   ├── torchserve/CLAUDE.md   # MAR files, custom handlers, management API
│   │   ├── ray-serve/CLAUDE.md    # Deployment, autoscaling, serialization
│   │   ├── bentoml/CLAUDE.md      # Bento packaging, runners, API security
│   │   ├── mlflow/CLAUDE.md       # Model registry, experiment tracking, artifacts
│   │   └── modal/CLAUDE.md        # Serverless functions, secrets, containers
│   │
│   └── frontend/                   # Frontend framework rules
│       ├── react/CLAUDE.md        # XSS prevention, state management, CSRF, forms
│       ├── nextjs/CLAUDE.md       # Server Components, Server Actions, middleware, env vars
│       ├── vue/CLAUDE.md          # v-html, computed properties, Vuex, router guards
│       ├── angular/CLAUDE.md      # DomSanitizer, template injection, HTTP client
│       └── svelte/CLAUDE.md       # {@html}, stores, SSR, form actions
│
├── templates/                      # Templates for adding new rules
│   ├── rule-template.md           # Template for individual rules
│   └── framework-template.md      # Template for framework rule sets
│
├── docs/                           # Documentation and guides
│   └── CONTRIBUTING.md            # Contribution guidelines with quality standards
│
├── compliance/                     # Compliance mapping (future)
│
├── CLAUDE.md                       # This file - project instructions
├── README.md                       # User documentation and implementation guide
└── LICENSE                         # MIT License
```

## Rule Counts

| Category | Count | Description |
|----------|-------|-------------|
| Core Rules | 4 | OWASP 2025, MCP Security, AI Security, Agent Security |
| Languages | 12 | Python, JavaScript, TypeScript, Go, Rust, Java, C#, Ruby, R, C++, Julia, SQL |
| Backend Frameworks | 5 | FastAPI, Express, Django, Flask, NestJS |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TikiTribe/claude-secure-coding-rules](https://github.com/TikiTribe/claude-secure-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
