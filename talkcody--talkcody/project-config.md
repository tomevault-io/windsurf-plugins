---
trigger: always_on
description: TalkCody 1.0 adopts a unified AI Agent architecture that supports both **Local** and **Service** deployment modes. All UI clients (Desktop, Mobile, Web, CLI, IM bots) function as thin interaction layers, communicating with the backend Agent service through a unified interface.
---

# Project Development Guide for AI Agents

## Project Overview

TalkCody 1.0 adopts a unified AI Agent architecture that supports both **Local** and **Service** deployment modes. All UI clients (Desktop, Mobile, Web, CLI, IM bots) function as thin interaction layers, communicating with the backend Agent service through a unified interface.

## Architecture Diagram

```
+----------------------------------------------------------------------------------+
|                                 UI / CLIENT LAYER                                |
+----------------------------------------------------------------------------------+
|  +----------+  +----------+  +----------+  +----------+  +----------+  +--------+ |
|  | Desktop  |  | Mobile   |  |   Web    |  |   CLI    |  |  IM Bots |  |  Other | |
|  | (mac/lnx)|  | iOS/Andr |  | Browser  |  |  TTY     |  | Feishu/  |  |  SDKs  | |
|  | Windows  |  |          |  |          |  |          |  | TG/Slack |  |        | |
|  +----------+  +----------+  +----------+  +----------+  +----------+  +--------+ |
+----------------------------------------------------------------------------------+
        |                                    |
        |                                    |
   +----+----+                          +----+----+
   |  LOCAL  |                          | SERVICE |
   |  MODE   |                          |  MODE   |
   | (Device)|                          | (Cloud) |
   +----+----+                          +----+----+
        |                                    |
        +----------------+-------------------+
                         |
                         v
+----------------------------------------------------------------------------------+
|                                  AGENT GATEWAY                                   |
|  +------------------+  +------------------+  +------------------+  +-----------+ |
|  | Auth & Tenancy   |  | Routing & Policy |  | Rate Limit & QoS |  | API Mgmt | |
|  +------------------+  +------------------+  +------------------+  +-----------+ |
|  +------------------+  +------------------+  +------------------+              |
|  | Session Manager  |  | Context Broker   |  | Tool Registry    |              |
|  +------------------+  +------------------+  +------------------+              |
+----------------------------------------------------------------------------------+
                                       |
                                       v
+----------------------------------------------------------------------------------+
|                              AI AGENT FRAMEWORK (Rust)                           |
|                                                                                  |
|   +--------------------+     +--------------------+     +--------------------+   |
|   |       TOOLS        |     |       SKILLS       |     |      MEMORY        |   |
|   | (exec, file, db)   |     | (domain modules)   |     | (short/long term)   |   |
|   +--------------------+     +--------------------+     +--------------------+   |
|                                                                                  |
|   +--------------------+     +--------------------+     +--------------------+   |
|   |      CONTEXT       |     |   OBSERVABILITY    |     |        LLM         |   |
|   | (state & buffers)  |     | (logs/metrics)     |     | (local/service)    |   |
|   +--------------------+     +--------------------+     +--------------------+   |
|                                                                                  |
|                         +------------------------------+                         |
|                         |          AGENT LOOP          |                         |
|                         |  Plan -> Act -> Observe ->   |                         |
|                         |  Reflect -> Update Context   |                         |
|                         +------------------------------+                         |
|                                                                                  |
+----------------------------------------------------------------------------------+
```

Important Notes:

The entire project is cur$$rently undergoing a refactoring process. New code for both the AI ​​agent (AGENT GATEWAY) and AI agent (FRAMEWORK) needs to be implemented in Rust.

## Technology Stack

### Frontend

- Framework: React 19
- Language: TypeScript
- Styling: Tailwind CSS + Shadcn UI
- Testing: Vitest
- State Management: Zustand

### Backend

- Framework: Tauri 2
- Language: Rust

### Database

- Type: SQLite

## Project Structure

```
talkcody/
│
├── src/                    # Main source code directory
│   ├── components/         # React UI components
│   ├── hooks/              # Custom React hooks
│   ├── lib/                # Utility functions and shared logic
│   ├── pages/              # Page components
│   ├── styles/             # Global styles and CSS
│   └── types/              # TypeScript type definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talkcody/talkcody](https://github.com/talkcody/talkcody) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
