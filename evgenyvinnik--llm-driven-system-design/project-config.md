---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Table of Contents

- [Repository Overview](#repository-overview)
- [Quick Start for Any Project](#quick-start-for-any-project)
- [Project Structure](#project-structure)
  - [Writing Architecture Documents](#writing-architecture-documents)
  - [Writing System Design Answers](#writing-system-design-answers)
  - [Explaining Trade-offs in Depth](#explaining-trade-offs-in-depth)
- [Common Commands](#common-commands)
- [Repository Scripts](#repository-scripts)
- [CI/CD](#cicd)
- [Technology Stack Defaults](#technology-stack-defaults)
- [Key Design Principles](#key-design-principles)
- [Port Conventions](#port-conventions)
- [Frontend Best Practices](#frontend-best-practices)
- [Backend Architecture Pattern](#backend-architecture-pattern)
- [Database Selection Guide](#database-selection-guide)
- [Creating New Projects](#creating-new-projects)
- [Local Development Philosophy](#local-development-philosophy)
- [Infrastructure Options](#infrastructure-options)
- [ESM Import Convention](#esm-import-convention)

## Repository Overview

This is a **system design learning repository** where each subdirectory represents an independent system design challenge. Most projects have both design documentation and working implementations.

**Node.js Requirement:** >=20.0.0

## Quick Start for Any Project

1. Read the project's `architecture.md` first to understand the design
2. Check `AGENTS.md` for iteration history and key decisions (primary source of truth for project-specific guidance - captures the "why" behind implementation choices)
3. Look at `README.md` for setup instructions
4. Check `package.json` to find available scripts

## Project Structure

Each project folder typically contains:
```
<project>/
├── README.md                  # Setup instructions and implementation guide
├── architecture.md            # System design documentation and trade-offs
├── system-design-answer-frontend.md   # Frontend-focused interview answer
├── system-design-answer-backend.md    # Backend-focused interview answer
├── system-design-answer-fullstack.md  # Full-stack interview answer
├── AGENTS.md                  # LLM collaboration notes and iteration history
├── frontend/                  # React + TypeScript frontend (when applicable)
├── backend/                   # Node.js + Express backend (when applicable)
├── training/                  # ML training code (Python, when applicable)
└── docker-compose.yml         # Infrastructure services (PostgreSQL, Redis, etc.)
```

**Documentation file purposes:**
- `architecture.md`: Dual-layer design document — production-scale architecture (the ideal) with diagrams, schemas, and trade-off analysis, plus local implementation notes (the actual) mapping to Docker + Node.js + React
- `system-design-answer-{frontend,backend,fullstack}.md`: Concise 45-minute interview answers using ASCII diagrams (no code blocks), trade-off tables, and first-person rationale. Each variant is tailored to a specific interview focus.
- `AGENTS.md`: Captures iteration history and the "why" behind key decisions

### Writing Architecture Documents

The `architecture.md` files serve a dual purpose: documenting the **production-scale design** (how this system would work at millions of users) and the **local implementation** (what we actually built with Docker + Node.js + Express + React). These two layers should be explicit, not interleaved.

**Recommended section structure:**

| Section | Scope | Purpose |
|---------|-------|---------|
| System Overview | Production | One-paragraph description + learning goals |
| Requirements | Production | Functional requirements + NFR targets at production scale (99.99% uptime, p99 < 50ms) |
| Capacity Estimation | Both | Optional. Production-scale estimates first; optional "Local Development Scale" subsection for component sizing. ~46% of projects include this |
| High-Level Architecture | Production | Box diagram showing the ideal architecture (CDN, API Gateway, microservices) |
| Core Components / Request Flows | Both | Component responsibilities and data flow at production scale, noting local simplifications inline |
| Database Schema | Both | Full SQL schemas with indexes and constraints — production-ready but run locally |
| API Design | Both | Endpoint listing with request/response examples |
| Key Design Decisions | Production | Trade-off analysis for major architectural choices |
| Consistency and Idempotency | Both | Idempotency keys, retry semantics, exactly-once guarantees |
| Security / Auth | Both | Authentication, authorization, rate limiting |
| Observability | Both | Prometheus metrics, structured logging, health checks |
| Failure Handling | Both | Circuit breakers, retry strategies, graceful degradation |
| Scalability Considerations | Production | Horizontal scaling path, sharding strategy, read replicas |
| Trade-offs Summary | Production | Summary table (see format below) |
| Implementation Notes | **Local** | Maps production design to local Docker + Node.js setup |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evgenyvinnik/llm-driven-system-design](https://github.com/evgenyvinnik/llm-driven-system-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
