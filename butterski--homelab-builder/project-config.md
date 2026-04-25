---
trigger: always_on
description: This document is the canonical reference for AI agents working on this codebase.
---

# HLBuilder — AI Agent Reference

This document is the canonical reference for AI agents working on this codebase.
It covers project architecture, testing infrastructure, known pitfalls, and the decisions behind them.

REMEMBER THAT THIS IS JUST TO BUILD AN MVP, NOT A PRODUCTION READY APP! 
SO YOU CAN WIPE THE DATABASE AND REBUILD IT FROM SCRATCH IF YOU WISH.
IF IT WOULD BE FASTER TO REBUILD THE APP FROM SCRATCH, THEN DO IT.
DO NOT MAKE ANY "todo later" OR "include in production" CHANGES.
EVEN THOUGH THIS IS NOT A PRODUCTION THIS NEEDS TO WORK LIKE IT.

Test things in docker since I wanna keep my windows environment clean.
Remember - I don't want migrations scripts or Legacy things support. If something needs to be changed in the database, just update the models and let the database be recreated.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Monorepo Layout](#monorepo-layout)
3. [Backend Architecture](#backend-architecture)
4. [HLBIPAM Microservice](#hlbipam-microservice)
5. [Frontend Architecture](#frontend-architecture)
6. [Data Model](#data-model)
7. [IP Assignment Algorithm](#ip-assignment-algorithm)
8. [Testing Infrastructure](#testing-infrastructure)
9. [Running Tests](#running-tests)
10. [Common Issues & Pitfalls](#common-issues--pitfalls)
11. [Fixed Bugs (Historical)](#fixed-bugs-historical)
12. [Environment Variables](#environment-variables)

---

## Project Overview

**HLBuilder** is a full-stack web app that lets users visually design their home lab network — placing hardware nodes (routers, switches, servers, NAS, etc.), wiring them, and automatically receiving IP address assignments and service recommendations.

- **Backend**: Go 1.24.5, Gin, GORM v1.31.1, PostgreSQL 17
- **HLBIPAM**: Standalone Go microservice for IP Address Management
- **Frontend**: React 18, TypeScript, Vite, ReactFlow, Zustand, Vitest
- **Infrastructure**: Docker Compose (postgres + backend + hlbipam + frontend)

---

## Monorepo Layout

```
homelab-builder/
├── docker-compose.yml          # postgres + backend + hlbipam + frontend services
├── docker-compose.test.yml     # test-specific compose overrides
├── Makefile                    # dev and test commands
├── AGENTS.md                   # this file
├── backend/
│   ├── cmd/
│   │   ├── server/main.go      # HTTP server entrypoint
│   │   └── migrate/main.go     # standalone migration runner
│   ├── internal/
│   │   ├── config/config.go    # env var loading
│   │   ├── handlers/           # Gin route handlers (one file per domain)
│   │   ├── middleware/         # auth, admin, rate limiter, security headers
│   │   ├── models/models.go    # ALL GORM models in one file
│   │   └── services/           # business logic; the only layer with tests
│   ├── migrations/             # raw SQL migrations (applied by postgres init)
│   ├── pkg/database/database.go
│   ├── go.mod
│   ├── Dockerfile              # multi-stage: builder → final scratch image
│   └── Dockerfile.test         # test runner image
├── hlbipam/                    # standalone IPAM microservice
│   ├── cmd/server/             # entrypoint
│   ├── internal/
│   │   ├── api/                # HTTP handlers
│   │   ├── core/               # allocator, subnet, types, validator
│   │   ├── models/             # data models
│   │   └── utils/              # utility functions
│   ├── Dockerfile
│   ├── Dockerfile.test
│   ├── go.mod
│   └── test_ipam.go            # integration test script
├── discord-bot/                # placeholder (empty)
├── frontend/
│   ├── src/
│   │   ├── features/           # domain-sliced feature modules
│   │   │   ├── admin/          # admin dashboard & management
│   │   │   ├── auth/           # authentication (Google OAuth, profile)
│   │   │   ├── builder/        # visual network builder (main feature)
│   │   │   ├── catalog/        # hardware & service catalog browsing
│   │   │   ├── donate/         # donation page
│   │   │   ├── landing/        # landing/login page
│   │   │   ├── setup-guide/    # setup checklist
│   │   │   ├── shopping/       # shopping list generation
│   │   │   └── survey/         # beta survey
│   │   ├── components/         # shared UI components
│   │   │   ├── auth/           # auth guards (RequireAuth)
│   │   │   ├── icons/          # icon components
│   │   │   ├── layout/         # sidebar, main layout
│   │   │   └── ui/             # design system primitives (button, dialog, etc.)
│   │   ├── lib/                # shared utilities
│   │   │   ├── api.ts          # base axios instance
│   │   │   ├── templates.ts    # config templates
│   │   │   └── utils.ts        # general utilities
│   │   ├── services/           # shared service layer (api.ts)
│   │   ├── types/index.ts      # shared TypeScript types
│   │   ├── App.tsx             # root component with routing
│   │   └── main.tsx            # React entry point
│   ├── vite.config.ts
│   └── package.json
└── docs/
    └── ARCHITECTURE.md         # copy of this file
```

---

## Backend Architecture

### Layers

```
HTTP Request → Gin Router → Middleware → Handler → Service → GORM → PostgreSQL
```

- **Handlers** (`internal/handlers/`): HTTP parsing, auth extraction, response shaping. No business logic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Butterski/homelab-builder](https://github.com/Butterski/homelab-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
