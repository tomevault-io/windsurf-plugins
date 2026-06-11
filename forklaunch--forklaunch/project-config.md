---
trigger: always_on
description: // CORRECT - Use framework packages
---

# Forklaunch Platform - Cursor AI Rules
# Auto-loaded by Cursor for AI assistance

## Critical Import Rules

### ✅ ALWAYS Import from @forklaunch Packages
```typescript
// CORRECT - Use framework packages
import { isRecord, camelCase, hashString } from '@forklaunch/common';
import { OpenTelemetryCollector } from '@forklaunch/core/http';
import { createCacheKey, TtlCache, TtlCacheRecord } from '@forklaunch/core/cache';
import { createObjectStoreKey, ObjectStore } from '@forklaunch/core/objectstore';
import { BaseEntity } from '@forklaunch/core/persistence';
import { SchemaValidator, string, number, uuid } from '@forklaunch/validator/zod';
import { forklaunchExpress, handlers } from '@forklaunch/express';
import { ForklaunchWebSocket, ForklaunchWebSocketServer } from '@forklaunch/ws';
import { RedisTtlCache } from '@forklaunch/infrastructure-redis';
import { S3ObjectStore } from '@forklaunch/infrastructure-s3';
import { TestContainerManager, BlueprintTestHarness } from '@forklaunch/testing';
```

### ❌ NEVER Import from @modules/core
```typescript
// WRONG - Don't do this
import { isRecord } from '@modules/core';  // NO!
import { OpenTelemetryCollector } from '@modules/core';  // NO!
```

## Import Organization (7 Layers)

Always organize imports in this order:

```typescript
// 1. Node built-ins (with node: prefix)
import crypto from 'node:crypto';
import path from 'node:path';

// 2. External dependencies
import { EntityManager } from '@mikro-orm/core';
import { injectable } from 'tsyringe';

// 3. Forklaunch framework packages
import { isRecord } from '@forklaunch/common';
import { OpenTelemetryCollector } from '@forklaunch/core/http';
import { forklaunchExpress } from '@forklaunch/express';
import { SchemaValidator } from '@forklaunch/validator/zod';

// 4. Cross-module imports
import { generateHmacAuthHeaders } from '@forklaunch-platform/iam';

// 5. Local persistence
import { Deployment } from '../../persistence/entities';

// 6. Local domain
import { DeploymentService } from '../services/deployment.service';

// 7. Same directory
import { EncryptionService } from './encryption.service';
```

## Project Structure

Follow this structure for all modules:

```
src/modules/<module-name>/
├── api/                     # HTTP interface
│   ├── controllers/        # Request handlers
│   ├── routes/            # Route definitions
│   ├── middleware/        # Middleware
│   └── utils/             # API utilities
├── domain/                 # Business logic
│   ├── services/          # Core business logic
│   ├── schemas/           # Validation schemas
│   ├── types/             # TypeScript types
│   ├── mappers/           # Entity ↔ DTO transformations
│   ├── enum/              # Enumerations
│   └── utils/             # Domain utilities
├── persistence/            # Data layer
│   ├── entities/          # Database models
│   └── seeders/           # Test data
├── migrations-postgresql/  # Database migrations
├── websocket/             # WebSocket handlers
├── registrations.ts       # Dependency injection
└── server.ts              # Entry point
```

## Mapper Usage Rules

### ✅ Use Mappers in Controllers
```typescript
// api/controllers/deployment.controller.ts
import { DeploymentMapper } from '../../domain/mappers/deployment.mappers';

export class DeploymentController {
  async get(req: Request, res: Response) {
    const deployment = await this.service.findById(req.params.id);
    // Use mapper for API response
    return res.json(DeploymentMapper.toDetailDto(deployment));
  }
}
```

### ❌ DON'T Use Mappers in Services
```typescript
// domain/services/deployment.service.ts
export class DeploymentService {
  // Return entities directly, NOT DTOs
  async findById(id: string): Promise<Deployment> {
    return this.em.findOne(Deployment, { id });
  }

  // Service-to-service calls use entities
  async process(deploymentId: string) {
    const deployment = await this.findById(deploymentId);
    await this.otherService.handle(deployment);  // Pass entity!
  }
}
```

### When to Use Mappers
- ✅ External API responses (controllers)
- ✅ Complex transformations
- ✅ Multiple API versions
- ❌ Internal service-to-service calls
- ❌ Simple pass-through data
- ❌ Single use cases

## CLI Command Usage

### ❌ NEVER Manually Edit Manifest
```bash
# WRONG
vim .forklaunch/manifest.toml
```

### ✅ ALWAYS Use CLI Commands
```bash
# Creating projects
forklaunch init service billing --database postgresql
forklaunch init worker email-worker --type bullmq
forklaunch init library shared-utils

# Modifying projects (ALWAYS use --dryrun first)
forklaunch change application --runtime bun --dryrun
forklaunch change application --runtime bun

forklaunch change service my-service --database postgresql --dryrun
forklaunch change service my-service --database postgresql

# Deployment
forklaunch release create --version 1.2.3
forklaunch deploy create --environment staging --region us-east-1
forklaunch deploy create --environment production --region us-east-1
```

## File Naming Conventions

```
Controllers:  <resource>.controller.ts     (deployment.controller.ts)
Services:     <resource>.service.ts        (deployment.service.ts)
Entities:     <resource>.entity.ts         (deployment.entity.ts)
Schemas:      <resource>.schema.ts         (deployment.schema.ts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forklaunch/forklaunch](https://github.com/forklaunch/forklaunch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
