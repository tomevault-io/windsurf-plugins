---
trigger: always_on
description: > AI Development Guide for Motia Projects
---

# AGENTS.md

> AI Development Guide for Motia Projects

This file provides context and instructions for AI coding assistants working on Motia projects.

## Project Overview

This is a **Motia** application - a framework for building event-driven, type-safe backend systems with:
- HTTP API endpoints (API Steps)
- Background event processing (Event Steps)  
- Scheduled tasks (Cron Steps)
- Real-time streaming capabilities
- Built-in state management
- Visual workflow designer (Workbench)

## Quick Start Commands

```bash
# Install dependencies
npm install

# Start development server (with hot reload)
npm run dev

# Start production server (without hot reload)
npm run start

# Generate TypeScript types from steps
npx motia generate-types
```

## 📚 Comprehensive Guides

**This project includes detailed Cursor rules in `.cursor/rules/` that contain comprehensive patterns and examples.**

These guides are written in markdown and can be read by any AI coding tool. The sections below provide quick reference, but **always consult the detailed guides in `.cursor/` for complete patterns and examples.**

### Available Guides

Read these files in `.cursor/rules/motia/` for detailed patterns:

- **`motia-config.mdc`** - Essential project setup, package.json requirements, plugin naming
- **`api-steps.mdc`** - Creating HTTP endpoints with schemas, validation, and middleware
- **`event-steps.mdc`** - Background task processing and event-driven workflows
- **`cron-steps.mdc`** - Scheduled tasks with cron expressions
- **`state-management.mdc`** - State/cache management across steps
- **`middlewares.mdc`** - Request/response middleware patterns
- **`realtime-streaming.mdc`** - WebSocket and SSE patterns
- **`virtual-steps.mdc`** - Visual flow connections in Workbench
- **`ui-steps.mdc`** - Custom visual components for Workbench

Architecture guides in `.cursor/architecture/`:

- **`architecture.mdc`** - Project structure, naming conventions, DDD patterns
- **`error-handling.mdc`** - Error handling best practices

**Read these guides before writing code.** They contain complete examples, type definitions, and best practices.

## Quick Reference

> **⚠️ Important**: The sections below are brief summaries. **Always read the full guides in `.cursor/rules/` for complete patterns, examples, and type definitions.**

### Project Structure

Motia discovers steps from both `/src` and `/steps` folders. Modern projects typically use `/src`:

**Recommended Structure (using `/src`):**
```
project/
├── .cursor/rules/   # DETAILED GUIDES - Read these first!
├── src/
│   ├── api/        # API endpoints
│   │   ├── users.step.ts
│   │   ├── orders.step.js
│   │   └── products_step.py
│   ├── events/     # Event handlers
│   │   ├── order-processing.step.ts
│   │   └── notifications_step.py
│   ├── cron/       # Scheduled tasks
│   │   └── cleanup.step.ts
│   ├── services/   # Business logic
│   ├── repositories/ # Data access
│   └── utils/      # Utilities
├── middlewares/    # Reusable middleware
│   └── auth.middleware.ts
├── motia.config.ts # Motia configuration
└── types.d.ts      # Auto-generated types
```

**Alternative Structure (using `/steps`):**
```
project/
├── steps/          # Step definitions
│   ├── api/
│   ├── events/
│   └── cron/
├── src/
│   ├── services/
│   └── utils/
└── motia.config.ts
```

### Step Naming Conventions

**TypeScript/JavaScript:** `my-step.step.ts` (kebab-case)  
**Python:** `my_step_step.py` (snake_case)

See `.cursor/architecture/architecture.mdc` for complete naming rules.

### Creating Steps - Quick Start

Every step needs two exports:

1. **`config`** - Defines type, routing, schemas, emits
2. **`handler`** - Async function with processing logic

**For complete examples and type definitions, read:**
- `.cursor/rules/motia/api-steps.mdc` - HTTP endpoints
- `.cursor/rules/motia/event-steps.mdc` - Background tasks
- `.cursor/rules/motia/cron-steps.mdc` - Scheduled tasks

## Detailed Guides by Topic

> **📖 Read the cursor rules for complete information**

### Step Types
- **API Steps** → Read `.cursor/rules/motia/api-steps.mdc`
  - HTTP endpoints, schemas, middleware, emits
  - Complete TypeScript and Python examples
  - When to use emits vs direct processing

- **Event Steps** → Read `.cursor/rules/motia/event-steps.mdc`
  - Background processing, topic subscriptions
  - Retry mechanisms, error handling
  - Chaining events for complex workflows

- **Cron Steps** → Read `.cursor/rules/motia/cron-steps.mdc`
  - Scheduled tasks with cron expressions
  - Idempotent execution patterns
  - Integration with event emits

### Architecture
- **Project Structure** → Read `.cursor/architecture/architecture.mdc`
  - File organization, naming conventions
  - Domain-Driven Design patterns (services, repositories)
  - Code style guidelines for TypeScript, JavaScript, Python

- **Error Handling** → Read `.cursor/architecture/error-handling.mdc`
  - ZodError middleware patterns
  - Logging best practices
  - HTTP status codes

### Advanced Features
- **State Management** → Read `.cursor/rules/motia/state-management.mdc`
  - Caching strategies, TTL configuration
  - When to use state vs database
  - Complete API reference

- **Middlewares** → Read `.cursor/rules/motia/middlewares.mdc`
  - Authentication, validation, error handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srujan-07/Schedviz](https://github.com/srujan-07/Schedviz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
