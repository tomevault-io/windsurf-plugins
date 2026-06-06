---
trigger: always_on
description: This is a production-ready backend HTTP service template designed for use with Claude. It provides a complete foundation for building Express.js applications with TypeScript, PostgreSQL, Docker, and comprehensive testing.
---

# Claude Template - Backend HTTP Service

This is a production-ready backend HTTP service template designed for use with Claude. It provides a complete foundation for building Express.js applications with TypeScript, PostgreSQL, Docker, and comprehensive testing.

**🌐 Live Demo:** https://claude-template-api.onrender.com
- **Health Check**: https://claude-template-api.onrender.com/health
- **API Docs**: https://claude-template-api.onrender.com/api-docs
- **Try it**: `curl https://claude-template-api.onrender.com/todo/list`

## 🚀 Quick Start

### Local Development
**Prerequisites:** Docker and Docker Compose

```bash
# 1. Start all services
npm run docker:up

# 2. Run database migrations
npm run docker:migrate

# 3. Seed with test data (optional)
npm run docker:seed

# 4. Test the API
curl http://localhost:3000/health
curl http://localhost:3000/todo/list
```

**Local Access Points:**
- **API Server**: http://localhost:3000
- **Swagger Docs**: http://localhost:3000/api-docs
- **pgAdmin**: http://localhost:5050 (admin@example.com / admin123)
- **Database**: PostgreSQL on port 5432

### Production Deployment (Render)
This template includes complete Render.com deployment configuration:

```bash
# 1. Push to GitHub
git push origin main

# 2. Connect to Render.com
# - Create new Blueprint from your repo
# - Render auto-detects render.yaml configuration
# - Creates PostgreSQL database + web service

# 3. Your API goes live automatically!
```

## Architecture Overview

This template follows an action-based API design pattern rather than traditional RESTful conventions. Key architectural decisions:

- **Action-based endpoints**: Uses explicit action names (e.g., `/todo/create`, `/todo/remove`) instead of REST verbs
- **Interactors pattern**: Business logic is organized in interactors, not controllers or services
- **Serializers**: Dedicated layer for response formatting and data transformation
- **TypeScript-first**: Full TypeScript support with strict mode enabled
- **Docker-first**: Complete containerized development environment
- **Test-driven**: Co-located test files with Jest configured

## Project Structure

```
src/
  routes/         # Express route definitions (action-based endpoints)
  interactors/    # Business logic layer (all domain logic lives here)
  serializers/    # Response formatting and data transformation
  models/         # Sequelize models for PostgreSQL
  middleware/     # Express middleware (error handling, validation, auth)
  config/         # Configuration files (database, swagger, app settings)
  utils/          # Utility functions and helpers
  migrations/     # Database migrations
  seeders/        # Database seed files
```

## Key Technologies

- **Runtime**: Node.js (latest version) with CommonJS
- **Framework**: Express.js with TypeScript
- **Database**: PostgreSQL with Sequelize ORM
- **Containerization**: Docker & Docker Compose
- **Testing**: Jest with co-located test files
- **Documentation**: Swagger/OpenAPI with decorators
- **Development**: tsx watch for hot reloading
- **Code Quality**: ESLint, Prettier, TypeScript strict mode

## API Design Philosophy

This template uses **action-based endpoints** instead of RESTful design:

```typescript
// Instead of RESTful:
// PUT /todos/:id
// DELETE /todos/:id

// We use action-based:
// POST /todo/create
// POST /todo/remove
// POST /todo/complete
// POST /todo/assign
```

This makes APIs more explicit and easier to understand.

## Docker Commands (Recommended)

```bash
# Service Management
npm run docker:up        # Start all services (PostgreSQL + app)
npm run docker:down      # Stop all services
npm run docker:logs      # View application logs
npm run docker:build     # Rebuild Docker images

# Database Operations  
npm run docker:migrate   # Run database migrations
npm run docker:seed      # Seed database with test data

# Development (if not using Docker)
npm run dev             # Start development server with hot reload
npm run build           # Build TypeScript to JavaScript
npm run test            # Run Jest tests
npm run test:watch      # Run tests in watch mode
npm run lint            # Run ESLint
npm run typecheck       # Run TypeScript type checking
```

## Testing Strategy

- Tests are co-located with source files (e.g., `create-todo.interactor.test.ts`)
- Unit tests for interactors focus on business logic
- Integration tests for routes test the full request/response cycle
- Use `supertest` for API endpoint testing
- Database is mocked or uses a test database

## Environment Variables

Create a `.env` file based on `.env.example`:

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
JWT_SECRET=your-secret-key
```

## Adding New Features

1. **Create the model** in `src/models/`
2. **Create interactors** in `src/interactors/[domain]/` with tests
3. **Create serializers** in `src/serializers/`
4. **Define routes** in `src/routes/` with action-based endpoints
5. **Add Swagger documentation** using decorators
6. **Create migrations** if database schema changes

## Code Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wSnarski/claude-template](https://github.com/wSnarski/claude-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
