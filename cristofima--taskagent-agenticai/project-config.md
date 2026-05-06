---
trigger: always_on
description: **Common Commands** (from repository root):
---

# Task Agent - Copilot Instructions

## Quick Reference

**Common Commands** (from repository root):

```powershell
# Backend (with Aspire Dashboard - recommended)
dotnet run --project src/TaskAgent.AppHost

# Backend (standalone)
cd src/backend/services/TaskAgent/src
dotnet run --project TaskAgent.WebApi

# Frontend
cd src/frontend/task-agent-web
pnpm install  # First time only
pnpm dev

# EF Migrations (from src/backend/services/TaskAgent/src/)
dotnet ef migrations add MigrationName --context TaskDbContext --project TaskAgent.Infrastructure --startup-project TaskAgent.WebApi --output-dir Migrations/TaskDb
dotnet ef database update --context TaskDbContext --project TaskAgent.Infrastructure --startup-project TaskAgent.WebApi

# Testing
cd src/backend/services/TaskAgent      # Backend tests (132 tests)
dotnet test                            # All tests
dotnet test --filter "UnitTests"       # Unit tests only

cd src/frontend/task-agent-web         # Frontend tests (108 tests)
pnpm test:run                          # Vitest unit tests (71)
pnpm e2e                               # Playwright E2E tests (37)
```

**Key URLs**:
- Frontend: http://localhost:3000
- Backend API: https://localhost:5001
- Aspire Dashboard: https://localhost:17198
- AG-UI endpoint: https://localhost:5001/agui

## Architecture Overview

AI-powered task management with **Microsoft Agent Framework** (preview), Clean Architecture, and dual-database persistence.

**Tech Stack**: .NET 10, Azure OpenAI (GPT-4o-mini), EF Core, .NET Aspire 13.0.2 | Next.js 16, React 19, TypeScript

**Clean Architecture** (strict downward dependencies):

```
src/backend/services/TaskAgent/src/
├── TaskAgent.Domain/           # Entities, Enums, Constants (NO dependencies)
├── TaskAgent.Application/      # DTOs, Interfaces, Functions (TaskFunctions.cs)
├── TaskAgent.Infrastructure/   # EF Core, Repositories, Azure Services
└── TaskAgent.WebApi/           # Controllers, AG-UI setup, DI configuration
```

## Critical Patterns

### 1. Domain Entity Factory Methods
```csharp
// ❌ NEVER: new TaskItem() { Title = "..." }
// ✅ ALWAYS: TaskItem.Create(title, description, priority)
var task = TaskItem.Create("My Task", "Description", TaskPriority.High);

// Business rules in entity methods
task.UpdateStatus(TaskStatus.InProgress);  // Validates transitions internally
```
See: `Domain/Entities/TaskItem.cs`

### 2. Scoped Dependencies in Singleton Agent
```csharp
// TaskFunctions is used by singleton AIAgent, but needs scoped DbContext
// ✅ Create scope per-call for fresh DbContext
using IServiceScope scope = _serviceProvider.CreateScope();
var repository = scope.ServiceProvider.GetRequiredService<ITaskRepository>();
```
See: `Application/Functions/TaskFunctions.cs`

### 3. Function Tools Contract
All methods in `TaskFunctions.cs` must:
- Use `[Description]` attributes for AI understanding
- Return user-friendly strings (✅/❌ emojis) - **NEVER throw exceptions**
- Catch all exceptions, return formatted error strings

### 4. PostgreSQL JSON Storage
```csharp
// ✅ Use json (preserves property order for $type discriminator)
entity.Property(e => e.SerializedThread).HasColumnType("json");
// ❌ NEVER jsonb (reorders properties alphabetically, breaks polymorphic deserialization)

// ✅ Use GetRawText() to preserve exact JSON structure
string serialized = threadJson.GetRawText();
// ❌ NEVER JsonSerializer.Serialize(threadJson) - reorders properties
```

## Dual Database Architecture

- **SQL Server** (`TaskDbContext`) - Task entities (structured CRUD)
- **PostgreSQL** (`ConversationDbContext`) - Conversation threads (JSON blob storage)

Both databases **MUST be available** on startup (fail-fast strategy).

## SSE Streaming & AG-UI

**Endpoints**:
- `POST /api/agent/chat` - Send message (SSE stream response)
- `GET /api/conversations` - List conversations
- `GET /api/conversations/{threadId}/messages` - Get history
- `DELETE /api/conversations/{threadId}` - Delete conversation

**SSE Event Types**: `STEP_STARTED`, `STATUS_UPDATE`, `STEP_FINISHED`, `TEXT_MESSAGE_START`, `TEXT_MESSAGE_CONTENT`, `TEXT_MESSAGE_END`, `TOOL_CALL_START`, `TOOL_CALL_RESULT`, `CONTENT_FILTER`, `THREAD_STATE`

**Dynamic Status Messages**: `FunctionDescriptionProvider` generates status from `[Description]` attributes (multi-agent ready, no hardcoded mappings).

**Content Safety**: Azure OpenAI's built-in filtering → Backend catches `ClientResultException` → Sends `CONTENT_FILTER` SSE event → Frontend displays friendly message in chat.

## Frontend Patterns

### SSE Streaming Client
```typescript
// ✅ Use chat-service.ts for all backend communication
import { sendMessage, listConversations } from "@/lib/api/chat-service";

// ✅ StreamingCallbacks for progressive UI updates
await sendMessage(request, {
  onTextChunk: (delta, fullText) => setMessage(fullText),
  onToolCallStart: (name, id) => setLoading(true),
  onComplete: (state) => saveThreadState(state),
});
```
See: `frontend/lib/api/chat-service.ts`

### Component Organization
```
components/
├── chat/           # ChatInput, ChatMessage, ChatMessagesList
├── conversations/  # ConversationSidebar, ConversationList, ConversationItem
└── shared/         # LoadingIndicator, DeleteConfirmModal
```

## Anti-Patterns to Avoid

| ❌ Don't | ✅ Do Instead |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cristofima/TaskAgent-AgenticAI](https://github.com/cristofima/TaskAgent-AgenticAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
