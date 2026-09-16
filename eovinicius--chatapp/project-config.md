---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
# Restore, build, test
dotnet restore
dotnet build
dotnet test

# Run a specific test class
dotnet test --filter "FullyQualifiedName~ConversationTest"

# Run the API (Swagger at http://localhost:5000/swagger)
dotnet run --project .\app\Api\ChatApp.Api\ChatApp.Api.csproj

# Start only the database (required before running locally)
docker-compose up -d --build chat-db

# EF migrations — one DbContext per module, each in its own schema.
# The design-time factories read DATABASE_CONNECTION_STRING, so no startup project is needed.
dotnet ef migrations add <Name> --project .\app\Modules\Chat\src\Chat.Infrastructure\
dotnet ef migrations add <Name> --project .\app\Modules\Identity\src\Identity.Infrastructure\
dotnet ef database update --project .\app\Modules\Chat\src\Chat.Infrastructure\
dotnet ef database update --project .\app\Modules\Identity\src\Identity.Infrastructure\
```

Do **not** pass `--no-build` when regenerating migrations after deleting the `Migrations/` folder — EF would read the stale snapshot from the previously compiled assembly and scaffold a diff instead of a clean initial migration.

## Architecture

**Modular Monolith** with Clean Architecture per module and **Minimal APIs** (no Controllers). The solution file `ChatApp.slnx` is at the repo root; code lives under `app/`:

- `app/Api/ChatApp.Api` — single host that only **composes** modules. No business logic; owns cross-cutting middleware (Serilog, CORS, versioning, Swagger, **rate limiter**, `ApplyMigrations` for both DbContexts). `AddIdentityModule` must be registered **before** `AddChatModule` — Identity owns the authentication scheme.
- `app/Modules/{Chat,Identity,Notification}` — each module has its own `src/` (four layers: `Domain`, `Application`, `Infrastructure`, `Presentation`) and `tests/`. **Chat** and **Identity** are implemented; Notification is still a scaffold.
- `app/Shared/SharedKernel` — `Result`, `Error`, `Entity`, `AggregateRoot`, `IDomainEvent`.
- `app/Shared/BuildingBlocks` — CQRS messaging (`ICommand`/`IQuery`/handlers), pipeline behaviors, `IDateTimeProvider`, `IUserContext`. No ASP.NET dependency.
- `app/Shared/BuildingBlocks.Api` — `ApiResults` (Result → HTTP) and `ValidationFilter<T>`, shared by every module's Presentation layer.

### Module boundary

**Identity owns users and authentication** (`identity` schema, `IdentityDbContext`): `User`, register/login/search, JWT issuing and validation, `IUserContext`.

**Chat never joins `identity.Users`** — it stores only `UserId` and resolves names/avatars through `Identity.Contracts`:

- `IUserDirectory.GetByIds(ids)` — batch hydration, used by the conversation list. Always batch; never call it per row.
- `IUserDirectory.Exists(userId)` — validating a DM target or new group members.
- `IUserPresenceSink.TouchLastSeen(userId, at)` — the hub reports disconnects; Identity persists "last seen".

Each module has its **own** unit-of-work type (`IUnitOfWork` for Chat, `IIdentityUnitOfWork` for Identity). They are deliberately distinct types: a single shared interface would make the two DI registrations collide.

Chat module layers (dependencies flow inward: **Presentation/API → Application → Domain**; Infrastructure implements Application interfaces):

- **Domain** (`Chat.Domain`): `Conversations/` (`Conversation`, `Participant`) and `Messages/` (`Message`, `MessageContent`, `ContentType`). Private constructors + static factories; every mutation that can fail returns `Result`.
- **Application** (`Chat.Application`): CQRS use cases under `UseCases/{Feature}/{UseCase}/`, plus abstractions (`IConversationDao`, `IChatNotifier`, `IPresenceTracker`, `IFileStorageService`).
- **Infrastructure** (`Chat.Infrastructure`): EF Core + PostgreSQL (`chat` schema), Dapper read models, SignalR, AWS S3.
- **Presentation** (`Chat.Presentation`): Minimal API endpoints dispatching via `ISender`; `ChatModule` exposes `AddChatModule`/`MapChatEndpoints` (which also maps the SignalR hub).

## Conversation model

`Conversation` has a `Type` — `Direct` (1x1) or `Group` — and the type governs every invariant:

- **Direct**: exactly two participants, immutable membership, no name and no admins. `DirectKey` is a deterministic ordered pair (`BuildDirectKey(a, b)`) with a **filtered unique index**, so opening a DM twice returns the same conversation. Attempts to add/remove/rename fail with `Conversation.DirectIsImmutable`.
- **Group**: name required, one `Owner`, `Admin`s, members join/leave. The owner cannot leave without `TransferOwnership` first.

`Participant` carries the **read/delivery cursors** (`LastReadMessageSentAt`, `LastDeliveredMessageSentAt`) — these hold the *`SentAt` of the last read message*, not the ack instant. That is what makes read receipts work without a message × user table:

- unread count = messages with `SentAt > LastReadMessageSentAt`
- ✓ sent · ✓✓ delivered (all others' delivery cursor ≥ `SentAt`) · ✓✓ blue (all others' read cursor ≥ `SentAt`)

Cursors are **monotonic** — an out-of-order ack from another device can never move them backwards.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eovinicius/chatapp](https://github.com/eovinicius/chatapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
