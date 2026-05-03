---
trigger: always_on
description: This project adopts **Monorepo + Vibe Coding Friendly DDD** architecture:
---

# Cursor AI Rules for Go-GenAI-Stack
# These rules help Cursor AI better understand and assist with developing this project

## Project Architecture

This project adopts **Monorepo + Vibe Coding Friendly DDD** architecture:

### Directory Structure
- `backend/`    - Backend (Go + Hertz + Eino + DDD)
- `frontend/`   - Frontend Monorepo
  - `web/`      - React Web application
  - `mobile/`   - React Native mobile application
  - `shared/`   - Frontend shared code (types, utils, constants)
- `scripts/`    - Project-level scripts
- `docs/`       - Project documentation

### Backend Architecture (Three-Layer DDD)

- Domain-First: Vertically split by business domain (domains/task/, domains/chat/, domains/llm/)
- Self-contained: Each domain contains model + service + repository + handlers + http + tests
- Explicit Knowledge: Each domain has 6 required files (README.md, glossary.md, rules.md, events.md, usecases.yaml, ai-metadata.json)

#### Three-Layer Architecture Details

```
domains/{domain}/
├── model/                 # 【Domain Model Layer】
│   └── {entity}.go        # Aggregate roots, entities, value objects
│                          # Responsibility: Simple domain rules and state changes
│
├── service/               # 【Domain Service Layer】⭐ Core
│   └── {domain}_service.go # Business logic implementation
│                          # Responsibility: Implement business use cases, encapsulate complex flows
│
├── repository/            # 【Repository Layer】
│   ├── interface.go       # Repository interface
│   └── {entity}_repo.go   # Repository implementation (database/sql)
│                          # Responsibility: Data persistence
│
├── handlers/              # 【HTTP Adapter Layer】
│   ├── dependencies.go    # Handler dependency container (inject Service)
│   └── {usecase}.handler.go # HTTP request handling
│                          # Responsibility: HTTP → Domain Input → HTTP
│
├── http/                  # 【HTTP Interface Layer】
│   ├── dto/              # Data Transfer Objects (close to API spec)
│   └── router.go         # Route registration
│
└── tests/                # 【Tests】
    └── {usecase}_test.go
```

**Dependency Direction**: Handler → Service → Repository → Database

**Core Principles**:
1. Handler layer (thin): Only HTTP adaptation, no business logic
2. Service layer (thick): Implement business use cases, encapsulate complex flows
3. Model layer: Simple domain rules and state changes
4. Repository layer: Data access abstraction

### Frontend Architecture

- Feature-First: Organized by feature (features/chat/, features/llm/)
- Domain Alignment: Frontend features align with backend domains (frontend/web/features/chat ← backend/domains/chat)
- Cross-platform Sharing: Web and Mobile share types, utils, constants via pnpm workspace

## Code Generation Rules

### 1. Adding New Use Case ⭐ Important

When the user requests to add a new business use case:

**Steps (Three-Layer Architecture)**:

1. **Read Explicit Knowledge**:
   - domains/{domain}/usecases.yaml (check use case definition)
   - domains/{domain}/README.md (understand domain boundaries)
   - domains/{domain}/glossary.md (understand terminology)
   - domains/{domain}/rules.md (understand business rules)
   - domains/{domain}/events.md (understand domain events)

2. **Generate Domain Service Layer** (core business logic):
   - File: `domains/{domain}/service/{domain}_service.go`
   - Method: Implement according to usecases.yaml steps order
   - Input/Output: Define dedicated Input/Output structs
   - Error handling: Use "ERROR_CODE: message" format
   - Example:
     ```go
     // service/task_service.go
     func (s *TaskService) CreateTask(ctx context.Context, input CreateTaskInput) (*CreateTaskOutput, error) {
         // Step 1: ValidateInput
         // Step 2: CreateTaskEntity
         // Step 3: SaveTask
         // Step 4: PublishEvent
     }
     ```

3. **Generate Handler Layer** (HTTP adaptation):
   - File: `domains/{domain}/handlers/{usecase}.handler.go`
   - Responsibility: HTTP request → Domain Input → Call Service → HTTP response
   - Example:
     ```go
     // handlers/create_task.handler.go
     func (deps *HandlerDependencies) CreateTaskHandler(ctx context.Context, c *app.RequestContext) {
         // 1. Parse HTTP request
         var req dto.CreateTaskRequest
         c.BindAndValidate(&req)
         
         // 2. Convert to Domain Input
         input := service.CreateTaskInput{...}
         
         // 3. Call Domain Service
         output, err := deps.taskService.CreateTask(ctx, input)
         
         // 4. Return HTTP response
         c.JSON(200, dto.CreateTaskResponse{...})
     }
     ```

4. **Generate DTO**:
   - File: `domains/{domain}/http/dto/{usecase}.go`
   - Include json tag, validation tag

5. **Generate Tests**:
   - File: `domains/{domain}/tests/{usecase}_test.go`
   - Cover all steps
   - Cover all error scenarios in errors list

6. **Update Documentation**:
   - Update README.md to add use case description

**Key Principles**:
- ✅ Business logic in Service layer
- ✅ Handler layer only does HTTP adaptation (thin layer)
- ✅ Each use case corresponds to one Service method
- ✅ Service Input/Output are pure business concepts (no HTTP details)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erweixin/Go-GenAI-Stack](https://github.com/erweixin/Go-GenAI-Stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
