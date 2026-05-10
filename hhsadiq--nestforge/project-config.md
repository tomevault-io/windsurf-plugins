---
trigger: always_on
description: project rules
---


# Cursor Rules

---

## 1. 🎯 Introduction & Purpose

**Mission**  
Enable **end-to-end development inside Cursor chat** with minimal context. Cursor should:

- Understand the boilerplate architecture.
- Enforce rules and structure from `docs/architecture.md`.
- Validate and generate Hygen JSON schemas.
- Run the correct Hygen commands automatically.
- Maintain adherence to hexagonal architecture.
- Guide developers through project creation inside Cursor.

---

## 2. 🧭 Core Principles & Naming Conventions

### Core Principles

- **Single source of truth**: Adhere to `docs/architecture.md`; don’t invent new patterns.
- **Type safety first**: Avoid `any`. Every public API must have explicit types.
- **Hexagonal architecture**: Services ↔ domain, repositories ↔ persistence, mappers handle conversions.
- **Small, focused changes**: Scope edits to one concern.
- **Cursor as assistant**: Act as a domain-aware guide—not a generic code generator.

### Naming & File Conventions

- **Classes / Types / Interfaces**: `PascalCase`
- **Variables / Functions / Methods**: `camelCase`
- **Files & Directories**: `kebab-case`
- **Environment Variables**: `UPPER_SNAKE_CASE`

Other guidelines:

- Avoid magic numbers—define constants.
- Boolean variables use verbs: `isLoading`, `hasError`, `canDelete`.

## 3. 🏗️ Architecture & Module Layout

Keep modules consistently structured:

### Parent Module Layout

```
src/[module]/
├── domain/[DOMAIN].ts
├── dto/
│   ├── create.dto.ts
│   ├── find-all.dto.ts
│   └── update.dto.ts
├── infrastructure/persistence/
│   ├── [PORT].repository.ts
│   ├── relational/
│   │   ├── entities/[ENTITY].ts
│   │   ├── mappers/[MAPPER].ts
│   │   ├── repositories/[ADAPTER].repository.ts
│   │   └── relational-persistence.module.ts
│   └── document/
├── controller.ts
├── service.ts
└── module.ts
```

### Sub-entity Structure

- Shares parent’s controller, service, module, and repository port/adapter.
- Adds:
  - `domain/[Child].ts`
  - `infrastructure/persistence/relational/entities/[child_entity].ts`
  - `infrastructure/persistence/relational/mappers/[child_mapper].ts`

---

## 4. 🧩 Hygen Generators

Cursor must know and run Hygen generators from `docs/hygen/`.

### Batch Generator

The batch generator supports comprehensive code generation from a single JSON file:

- **Supports**: entities, sub-entities, relationships, and enums independently or in combination
- **Input**: Array of objects in `.hygen-entities-generator/entities-generator.json`
- **Command**: `npm run generate:entities`
- **File**: `.hygen-entities-generator/index.js`
- **Samples**: `.hygen-sample-files/`

**Key Features:**

- Generates parent resources, sub-entities, relationships, and enums in correct order
- Supports partial input (only entities, only relationships, only enums, or any combination)
- Idempotent runs - skips existing files
- Automatic cleanup and linting after generation

#### SQL to JSON Conversion

When users provide SQL `CREATE TABLE` statements and ask to create JSON schemas, use the [SQL to JSON Entities Prompt](docs/hygen/sql-to-json-entities-prompt.md) to generate complex or simple JSON based on the provided SQL and use it in `.hygen-entities-generator/entities-generator.json`

### Individual Generators

Each generator supports both interactive prompts and JSON-driven generation:

**Multi-Support Generators** (Interactive + JSON):

- **Resource-entity**: `npm run generate:resource`
- **Sub-entity**: `npm run generate:sub-entity`
- **Relationship**: `npm run generate:relationship`
- **Enum**: `npm run generate:enum`

**Interactive-Only Generators** (Command prompts only):

- **Property**: `npm run add:property`
- **Raw Query**: `npm run generate:raw-query`
- **Version**: see `docs/hygen/version.md`

**JSON Usage**: When using JSON, generators accept a single JSON object via `DATA_FILE` environment variable.

### 📑 JSON Schemas & Validation

Cursor should validate/generate json objects based on following schema examples:

```jsonc
// Resource-Entity
{
  "parent": null,
  "name": "EntityName",
  "functionalities": ["create","findAll","findAllWithSearch","findOne","update","delete"],
  "fields": [{
        "name": "name",
        "type": "varchar",
        "customType": "",
        "optional": false,
        "example": "League",
        "dto": true,
        "associatedEnumName": "MatchCategory",
        "unique": true
      }],
  "relations": [], // may have values depending upon the requirements
  "enums": [] // may have values depending upon the requirements
}

// Sub-entity
{
  "parent": "ParentEntityName",
  "name": "ChildEntityName",
  "functionalities": ["create","findAll","findAllWithSearch","findOne","update","delete"],
  "fields": [{
        "name": "name",
        "type": "varchar",
        "customType": "",
        "optional": false,
        "example": "League",
        "dto": true,
        "associatedEnumName": "MatchCategory",
        "unique": true
      }],
  "enums": [], // may have values depending upon the requirements
  "relations": [] // may have values depending upon the requirements
}

// Relationship
{
  "sourceEntityName": "Coach",
  "sourceEntityParent": null,
  "relationType": "OneToOne",
  "relationEntityParent": null,
  "relationEntityName": "Team",
  "sourceColumnName": "team_id",
  "relationFieldName": null
}

// Enum
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hhsadiq/NestForge](https://github.com/hhsadiq/NestForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
