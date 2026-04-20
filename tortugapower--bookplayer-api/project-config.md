---
trigger: always_on
description: This document provides a comprehensive overview of the BookPlayer API backend for new developers.
---

# BookPlayer API - Developer Guide

This document provides a comprehensive overview of the BookPlayer API backend for new developers.

## Quick Start

```bash
# Install dependencies
yarn install

# Set up environment (copy and edit)
cp development.env.template .development.env

# Run in development mode (auto-reload)
yarn dev

# Server runs at http://localhost:5003
# Health check: GET /v1/status
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Express Application                       │
│      (Auth, CORS, Helmet, Compression, Version middlewares) │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                  Router Layer (/src/api/)                   │
│   UserRouter │ PasskeyRouter │ LibraryRouter │ AdminRouter  │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│             Controller Layer (/src/controllers/)            │
│    Handle HTTP request/response, call services              │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│              Service Layer (/src/services/)                 │
│    Business logic, database queries, external APIs          │
└────────────────────────────┬────────────────────────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
         PostgreSQL       Redis         External
          (Knex)        (Cache)      (AWS, Apple,
                                     RevenueCat)
```

## Project Structure

```
src/
├── main.ts                 # Entry point
├── server.ts               # Express app setup
├── container.ts            # Dependency injection bindings
├── ContainerTypes.ts       # DI symbols
├── config/
│   └── envs.ts            # Environment validation
├── api/
│   ├── RouterHttp.ts      # Main router (mounts all routes)
│   ├── UserRouter.ts      # /v1/user routes
│   ├── PasskeyRouter.ts   # /v1/passkey routes
│   ├── LibraryRouter.ts   # /v1/library routes
│   ├── AdminRouter.ts     # /v1/admin routes
│   ├── StorageRouter.ts   # /v1/storage routes
│   ├── RetentionMessagingRouter.ts  # /v1/retention routes
│   └── middlewares/
│       ├── auth.ts        # JWT validation
│       ├── version.ts     # App version check
│       ├── subscription.ts # Subscription check
│       └── admin.ts       # Admin permission check
├── controllers/           # HTTP handlers
├── services/              # Business logic
├── interfaces/            # TypeScript interfaces (I* prefix)
├── types/                 # Type definitions
├── database/
│   ├── index.ts          # Knex connection
│   └── migrations/       # Database migrations
└── utils/                # Shared utilities
```

## Dependency Injection (Inversify)

All components use Inversify for dependency injection.

### Adding a New Service

1. **Create the interface** (`src/interfaces/IMyService.ts`):
```typescript
export interface IMyService {
  DoSomething(param: string): Promise<Result>;
}
```

2. **Create the service** (`src/services/MyService.ts`):
```typescript
import { injectable, inject } from 'inversify';
import { TYPES } from '../ContainerTypes';
import { ILoggerService } from '../interfaces/ILoggerService';

@injectable()
export class MyService implements IMyService {
  @inject(TYPES.LoggerService)
  private _logger: ILoggerService;

  private db = database;

  async DoSomething(param: string): Promise<Result> {
    try {
      // Business logic here
      return result;
    } catch (err) {
      this._logger.log({
        origin: 'MyService.DoSomething',
        message: err.message,
        data: { param }
      });
      return null;
    }
  }
}
```

3. **Add symbol** (`src/ContainerTypes.ts`):
```typescript
const TYPES = {
  // ...existing types
  MyService: Symbol.for('MyService'),
};
```

4. **Register binding** (`src/container.ts`):
```typescript
import { IMyService } from './interfaces/IMyService';
import { MyService } from './services/MyService';

container.bind<IMyService>(TYPES.MyService).to(MyService);
```

5. **Inject in controller**:
```typescript
@injectable()
export class MyController {
  @inject(TYPES.MyService)
  private _myService: IMyService;
}
```

## Request Flow

### 1. Middleware Pipeline (server.ts)

```typescript
app.use(bodyParser.json());           // Parse JSON
app.use(compress());                   // Gzip compression
app.use(helmet());                     // Security headers
app.use(authMiddleware);              // JWT validation → sets req.user
app.use(versionMiddleware.check());   // Version compatibility
app.use('/v1', routerHttp.get());     // Mount routes
app.use(handleError);                 // Global error handler
```

### 2. Auth Middleware

The auth middleware (`src/api/middlewares/auth.ts`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TortugaPower) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
