---
trigger: always_on
description: **Node.js/Express/TypeScript Development Guide**
---


# AGENTS.md
**Node.js/Express/TypeScript Development Guide**

**Version:** 1.2.3  
**Status:** Canonical guide for AI-assisted Node/Express/TypeScript development  
**Last Updated:** February 2026

***

## Table of Contents

1. **Compliance Core Rules**
2. **Express/TypeScript Development Standards**
3. **Session Startup Context**
4. **Project Structure Memory Bank**
5. **OpenSpec Workflow**
6. **Task Management with OpenSpec**
7. **Quality & Testing**
8. **Memory Management**
9. **Security Code Review**
10. **Troubleshooting**

***

### Note on SKILLS/AGENTS
SKILLs and AGENTS live in .github/ for VSCode and .agents/ for Windsurf and can should be used when the work needs specific knowledge that a skill or agent contains 

### Note on MCP
**Always** check for tools and MCP servers to assist with modifications e.g. use the shadcn-UI-mcp to find default components rather than building from scratch.

***

## 1. Compliance Core Rules

### Startup Compliance Output (Every Session)

| Rule | Requirement | Validation |
|------|------------|-----------|
| ❌ **No new files without reuse analysis** | Search codebase, check existing controllers/services, provide justification | "Analyzed `src/controllers/X`, `src/services/Y`. Cannot extend because [reason]" |
| ❌ **No rewrites when refactoring possible** | Prefer incremental improvements to existing services/controllers | "Extending `User` service at line X rather than creating new service" |
| ❌ **No ignoring Express/TS conventions** | Follow Express REST patterns, layered architecture (routes → controllers → services → data access), TypeScript strict types, CRA conventions for React | "Follows Express patterns: routes in `src/routes/`, controllers in `src/controllers/`, services in `src/services/`, DB access in `src/db/` or `src/repositories/`; React follows CRA `src/` structure" |
| ❌ **No skipping tests** | TDD mandatory—red, green, refactor. Never commit without tests | "Red: wrote failing test \| Green: implemented \| Verified: exit code 0" |

### The Four Sacred Rules (Express/TypeScript-Specific)

- **Approval Gates**: No commits without explicit user approval
- **Sandbox First**: All work in feature branches, never `main`/`master`
- **Citations**: Always `src/path/file.ts:42` (single line) or `src/path/file.ts:42-58` (range)
- **No Mock Data**: Never fake/simulated data in production; test fixtures are OK
- **No Secrets**: Never hardcode API keys, passwords, or credentials
- **TDD Mandatory**: Tests written before production code
- **Code Quality**: Must pass `npm run lint`, `npm test`, and `ubs .` before commit
- **Task Tracking**: Use OpenSpec workflows for ALL work—no markdown TODOs

***

### Non-Negotiables

Following Express/TypeScript conventions reduces code. Always ask: **"Does Express/Node already provide this?"**

**Good:**
- **Route**: `src/routes/users.ts` defining endpoints
- **Controller**: `src/controllers/usersController.ts` handling requests/responses
- **Service**: `src/services/usersService.ts` containing business logic
- **Repository/DB**: `src/db/usersRepository.ts` or `src/repositories/usersRepository.ts` for data access
- **Model/Types**: `src/models/User.ts` or `src/types/User.ts` for TypeScript interfaces/types

**Anti-Pattern:**
- Custom database layer bypassing established patterns
- Non-standard directory structures
- Controllers with business logic (should be in services)
- Missing TypeScript types

***

## 2. Express/TypeScript Development Standards

### Core Express Principles

#### Layered Architecture (Separation of Concerns)

| Component | Responsibility | Anti-Pattern |
|-----------|---------------|--------------|
| **Route** | Define endpoints, map HTTP methods to controllers | Business logic in routes |
| **Controller** | Handle requests/responses, validate input, call services | Direct database queries, business logic |
| **Service** | Business logic, orchestration, data transformation | HTTP-specific code, direct DB queries |
| **Repository/Data Access** | Database operations, queries, transactions | Business logic |
| **Model/Type** | TypeScript interfaces/types, data shape validation | Logic implementation |

#### RESTful Design

Actions correspond to HTTP verbs:

```typescript
// src/routes/users.ts
import express from 'express';
import { usersController } from '../controllers/usersController';

const router = express.Router();

router.get('/', usersController.index);        // GET /users - list all
router.get('/:id', usersController.show);      // GET /users/:id - single resource
router.post('/', usersController.create);      // POST /users - create new
router.put('/:id', usersController.update);    // PUT /users/:id - full update
router.patch('/:id', usersController.update);  // PATCH /users/:id - partial update
router.delete('/:id', usersController.destroy);// DELETE /users/:id - delete

export default router;
```

#### TypeScript Strict Types

All code must use strict TypeScript:

```typescript
// src/types/User.ts
export interface User {
  id: number;
  email: string;
  name: string;
  createdAt: Date;
}

export interface CreateUserDTO {
  email: string;
  name: string;
}
```

Never use `any` without justification. Use proper types or `unknown` with type guards.

#### Thin Controllers, Logic in Services

**Good:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jatwell93) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
