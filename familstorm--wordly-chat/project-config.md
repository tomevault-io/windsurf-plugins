---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a NestJS-based backend application called "wordly_chat". It follows the standard NestJS architecture with TypeScript, using modern ES modules (nodenext) and decorators for dependency injection and routing.

## Development Commands

### Running the Application
- `npm run start` - Start the application in standard mode
- `npm run start:dev` - Start in watch mode (auto-reloads on file changes)
- `npm run start:debug` - Start in debug mode with watch enabled
- `npm run start:prod` - Run the production build from dist/

### Building
- `npm run build` - Compile TypeScript to JavaScript (output: dist/)
- The build uses the NestJS CLI and outputs to `./dist` directory

### Testing
- `npm run test` - Run all unit tests (*.spec.ts files in src/)
- `npm run test:watch` - Run tests in watch mode
- `npm run test:cov` - Run tests with coverage report
- `npm run test:e2e` - Run end-to-end tests (test/ directory)
- `npm run test:debug` - Run tests in debug mode with Node inspector

### Code Quality
- `npm run lint` - Run ESLint with auto-fix on all TypeScript files
- `npm run format` - Format code with Prettier

### Docker Commands
- `docker-compose up -d` - Start all services (chat-service, MongoDB, Redis) in detached mode
- `docker-compose up --build` - Rebuild and start all services
- `docker-compose down` - Stop and remove all containers
- `docker-compose down -v` - Stop containers and remove volumes (clears database data)
- `docker-compose logs -f chat-service` - View chat service logs
- `docker-compose logs -f mongo` - View MongoDB logs
- `docker-compose logs -f redis` - View Redis logs
- `docker-compose ps` - List all running containers
- `docker-compose exec chat-service sh` - Access chat service container shell
- `docker-compose exec mongo mongosh` - Access MongoDB shell
- `docker-compose exec redis redis-cli` - Access Redis CLI

## Architecture

### Module System
The application uses NestJS's modular architecture:
- **AppModule** (src/app.module.ts) - Root module that imports all feature modules
- Modules follow the pattern: Module → Controller → Service
- Each module declares its controllers and providers

### Entry Point
- **main.ts** - Application bootstrap, creates NestJS instance and starts HTTP server on port 3000 (or PORT env var)

### Dependency Injection
NestJS uses decorators for DI:
- `@Injectable()` - Marks classes as providers (services)
- `@Controller()` - Defines route controllers
- `@Module()` - Defines modules with imports, controllers, and providers
- Services are injected via constructor parameters

### TypeScript Configuration
- Uses `nodenext` module system with ES modules
- Decorators enabled (`experimentalDecorators`, `emitDecoratorMetadata`)
- Target: ES2023
- Strict null checks enabled, but `noImplicitAny` disabled
- Output directory: `./dist`

### Testing
- Unit tests: Co-located with source files (*.spec.ts)
- E2E tests: Separate test/ directory with jest-e2e.json config
- Jest is configured with ts-jest for TypeScript support
- Test root: src/ directory

### Configuration Management
The application uses a centralized configuration system with Joi validation:

- **ConfigurationModule** (src/config/config.module.ts) - Global configuration module
- **AppConfigService** (src/config/app-config.service.ts) - Type-safe configuration access
- **Joi Validation** - Environment variables validated at startup
- **Namespaced Config** - Organized by domain (app, database, redis, auth)

#### Configuration Access Pattern
```typescript
constructor(private readonly configService: AppConfigService) {}

// Type-safe access
const port = this.configService.app.port;
const dbUri = this.configService.database.uri;
const redisHost = this.configService.redis.host;
const jwtSecret = this.configService.auth.jwtSecret;

// Environment checks
if (this.configService.isProduction) { ... }
```

#### Required Environment Variables
- MongoDB: `MONGO_DATABASE`, `MONGO_USERNAME`, `MONGO_PASSWORD`
- Redis: `REDIS_HOST`, `REDIS_PASSWORD`
- Auth: `JWT_SECRET`, `JWT_REFRESH_SECRET`, `SESSION_SECRET` (min 32 chars each)

See `.env.example` for complete list and `src/config/README.md` for detailed documentation.

## Docker & Infrastructure

The application uses Docker Compose to orchestrate three services:

### Services
1. **chat-service** - NestJS application container
   - Multi-stage build (builder + production)
   - Runs as non-root user (nestjs:nodejs)
   - Exposed on port 3000 (configurable via PORT env var)
   - Depends on MongoDB and Redis health checks

2. **mongo** - MongoDB 7.0 database
   - Persistent storage via Docker volumes
   - Root credentials configurable via environment variables
   - Health check using mongosh ping
   - Port: 27017

3. **redis** - Redis 7 cache/message broker
   - Password-protected (configurable)
   - Persistent storage via Docker volumes
   - Health check using redis-cli
   - Port: 6379

### Environment Variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [familstorm/wordly_chat](https://github.com/familstorm/wordly_chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
