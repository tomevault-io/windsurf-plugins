---
trigger: always_on
description: **CRITICAL SYSTEM DIRECTIVE FOR AI AGENTS**:
---

# FivFold CLI Refactoring & Architectural Rule Set (AGENTS.md)

**CRITICAL SYSTEM DIRECTIVE FOR AI AGENTS**:
You are tasked with refactoring and extending the FivFold CLI. FivFold exists as two CLIs (`@fivfold/ui` and `@fivfold/api`) built on a shared engine (`@fivfold/core`). The core already implements VFS, StrategyPipeline, manifests, and AST (ts-morph). Your objective is to extend this into a highly advanced, extensible, multi-stack scaffolding engine as defined below.

You must strictly adhere to these rules. Any code generated that violates these constraints is considered a failure.

## 0. Strict Tech Stack Constraints

Do not use legacy versions of these tools. The project requires:

- **Node.js:** v20 or later.
- **Frontend:** React 18+ or Next.js 14+ (App Router).
- **Styling:** Tailwind CSS **v4 exclusively** (using `@import "tailwindcss"`; NO `tailwind.config.js` is permitted).
- **UI Foundation:** shadcn/ui.

## 1. File Modification Rules (The "Hybrid" Strategy)

**NEVER** use Regular Expressions or string `.replace()` functions to mutate existing source code files (like `app.module.ts` or `server.ts`). This is strictly forbidden due to fragility.

- **For EXISTING files:** You MUST use Abstract Syntax Tree (AST) manipulation via `ts-morph`. You must parse the AST, safely inject the node, and serialize it back to ensure surgical precision and avoid destroying user modifications.
- **For NEW, standalone files:** You may use string-based templating (e.g., Handlebars).

## 2. State & Transaction Rules (Virtual File System)

**NEVER** write directly to the physical disk (using `fs.writeFileSync`) in the middle of a generation process.

- **VFS Mandate:** You must implement a Virtual File System (VFS). All file creations, AST modifications, and deletions must be staged in memory first.
- **Atomic Commits:** Only after all generators and AST mutations complete successfully should the VFS flush changes to the disk in a single, atomic transaction.
- **Side Effects:** Actions like `npm install` or running formatting tools must only occur *after* the VFS successfully commits to the disk.
- **Dry Run:** The system must support a `--dry-run` flag to output intended changes without executing them.

**Current implementation:** `@fivfold/core` provides `VirtualFileSystem` with `stageCreate`, `stageModify`, `stageDelete`, `preview`, and `commit`.

## 3. Anti-Combinatorial Explosion Rules

**NEVER** create hardcoded directories for every permutation of a stack (e.g., no `express-typeorm-firebase` folders).

- **Strategy Pattern:** Implement interchangeable classes for code generation (e.g., `NestJsGeneratorStrategy`, `MongooseOrmStrategy`) instead of massive `if/else` statements.
- **Manifests over Hardcoding:** Kits must be defined by declarative JSON/YAML schemas (Manifests) specifying dependencies, remote template URLs, and AST mutation targets. The CLI must act as an agnostic orchestrator that reads these manifests.
- **ORM-Agnostic Framework Layer:** The `framework` manifest layer (Express/NestJS files and deps) must **never** hardcode ORM-specific dependencies (e.g., `typeorm`, `pg`, `reflect-metadata`). Those belong exclusively in the `orm` manifest layer, supplied by the active ORM strategy.
- **Handlebars Conditionals for ORM Imports:** Framework templates (`*.service.ts.hbs`, `*.module.ts.hbs`) must use `{{#if (eq orm "typeorm")}}` / `{{#if (eq orm "mongoose")}}` / etc. conditionals for ORM-specific imports, decorators, and dependency injection patterns.

**Current implementation:** `@fivfold/core` provides `StrategyPipeline`, `IGeneratorStrategy`, `IRealtimeStrategy`, and registry. API package uses `DomainStrategy`, `TypeOrmOrmStrategy`, `PrismaOrmStrategy`, `MongooseOrmStrategy`, `CosmosOrmStrategy`, `DynamoDbOrmStrategy`, `NestJsFrameworkStrategy`, `ExpressFrameworkStrategy`, `PushProviderStrategy`, `RealtimeProviderStrategy`. Manifests live in `ui/manifests/` and `api/manifests/`.

## 4. Output Architecture Rules (Hexagonal/Ports & Adapters)

When scaffolding backend code (like an Auth Kit), the outputted code **MUST** adhere to Hexagonal Architecture to prevent vendor lock-in.

- **Domain (Core):** Must generate framework-agnostic Interfaces (Ports) (e.g., `IAuthService`, `IChatService`).
- **Infrastructure (Adapters):** Must generate separate files implementing the Interfaces using vendor SDKs (e.g., `FirebaseAuthAdapter`, `MongooseChatAdapter`).
- **Delivery:** HTTP transport (Express Routes or NestJS Controllers) must be isolated from the core logic.

### Auth Provider Independence

Auth provider selection (Firebase Auth, Cognito, Auth0, JWT) is **completely independent** of database/ORM selection. A user may choose Firebase Authentication as their auth provider while storing application data in PostgreSQL with TypeORM, MongoDB with Mongoose, or any other supported combination. The auth provider handles identity verification only; all application data (user profiles, sessions, kit-specific data) lives in the user's chosen database.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fivex-Labs/fivfold](https://github.com/Fivex-Labs/fivfold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
