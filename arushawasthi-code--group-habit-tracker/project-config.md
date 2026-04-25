---
trigger: always_on
description: <!-- MAINTENANCE RULE: When you modify code in this project, check whether this
---

<!-- MAINTENANCE RULE: When you modify code in this project, check whether this
file still accurately reflects the codebase. If you added/removed/renamed files,
changed commands, added dependencies, or altered patterns — update this file in
the same commit. When reviewing code you didn't write, verify this file matches
reality before trusting it. -->

# HabitHive — Agent Reference

## Version Control

Repository: `git@github.com:arushawasthi-code/group-habit-tracker.git`

```bash
git status / git log --oneline   # standard workflow
git checkout -b fix/<topic>      # branch naming convention
git push -u origin <branch>      # push without creating a PR yet
```

**Branch convention:** `fix/<topic>` for bug/polish branches, `feat/<topic>` for new features. `master` is the stable baseline — never commit directly to it for anything non-trivial.

**`.gitignore` covers:** `bin/`, `obj/`, `*.db`, `node_modules/`, `dist/`, `test-results/`, `playwright-report/`.

---

## Commands

```bash
# API (run from src/HabitHive.Api/)
dotnet build
dotnet run --launch-profile http      # http://localhost:5000
dotnet run                            # uses default profile

# Frontend (run from src/habithive-client/)
npm install
npm run dev                           # http://localhost:5173
npm run build
npx tsc --noEmit                      # type-check only

# Migrations (run from src/HabitHive.Api/)
dotnet ef migrations add <MigrationName>   # create a new migration
dotnet ef migrations remove                # undo last migration (before it's applied)
# DB is auto-migrated at startup via db.Database.Migrate() in Program.cs
# To reset DB in dev: delete src/HabitHive.Api/habithive.db and restart
```

## Project Structure

```
/
├── HabitHive.sln
├── src/
│   ├── HabitHive.Api/
│   │   ├── Controllers/        REST endpoints (thin — no logic)
│   │   ├── Hubs/               SignalR ChatHub
│   │   ├── Models/             EF entities + Dtos.cs (all DTOs in one file)
│   │   ├── Data/               HabitHiveDbContext.cs
│   │   ├── Services/           All business logic
│   │   ├── wwwroot/uploads/    Local photo storage (served as static files)
│   │   ├── appsettings.json    JWT config + Tenor API key
│   │   └── Program.cs          DI, middleware, EF, SignalR, CORS
│   └── habithive-client/
│       └── src/
│           ├── components/     UI components
│           ├── hooks/          useAuth, useSignalR, jwtDecode
│           ├── pages/          LoginPage
│           ├── services/api.ts Axios client + all API calls
│           └── types/index.ts  All TypeScript types + enums
```

## Tech Stack

| Layer | Tech | Version |
|-------|------|---------|
| Backend runtime | .NET / ASP.NET Core | 10.0 |
| ORM | EF Core + Sqlite provider | 10.0.5 |
| Auth | Microsoft.AspNetCore.Authentication.JwtBearer | 10.0.5 |
| Password hashing | BCrypt.Net-Next | 4.1.0 |
| Real-time | ASP.NET Core SignalR | (built-in) |
| Frontend | React + TypeScript | 19 / 5.9 |
| Build tool | Vite + @tailwindcss/vite | 8.0 |
| CSS | Tailwind CSS v4 (vite plugin, no tailwind.config) | 4.x |
| HTTP client | Axios | 1.x |
| SignalR client | @microsoft/signalr | latest |
| Animations | canvas-confetti | latest |

## Key Patterns & Conventions

**Business logic location:** Services only (`AuthService`, `HabitService`, `GroupService`, `ChatService`). Controllers extract `UserId` from JWT claims and delegate — no logic in controllers.

**User ID extraction (controllers):**
```csharp
private Guid GetUserId() => Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);
```

**User ID extraction (SignalR hub):**
```csharp
private Guid GetUserId() => Guid.Parse(Context.User!.FindFirstValue(ClaimTypes.NameIdentifier)!);
```

**SignalR auth:** Token passed via `?access_token=` query string (not Authorization header). Configured in `JwtBearerEvents.OnMessageReceived` in `Program.cs`. The hub is `[Authorize]`-decorated.

**Chat message types:** `ChatMessage.Content` is either plain text or a JSON payload depending on `MessageType`. JSON is only parsed client-side for rendering — never queried server-side. All types share one table.

**Habit visibility:** Privacy via presence/absence of `HabitGroupVisibility` rows (composite PK `HabitId + GroupId`). No row = private. Visibility is set via `PUT /api/habits/{id}/visibility` with the full list of target `groupIds`.

**Group soft-delete:** `GroupMember.LeftAt` nullable DateTime. Active members = `LeftAt == null`. Filter pattern: `.Where(gm => gm.LeftAt == null)`.

**DB schema:** EF Core migrations in `src/HabitHive.Api/Migrations/`. `db.Database.Migrate()` in `Program.cs` applies pending migrations on startup. Add new migrations with `dotnet ef migrations add <Name>` from `src/HabitHive.Api/`.

**Streak calculation:** Lives in `HabitService.CalculateStreak()`. Uses UTC dates (`DateOnly`). Daily = consecutive calendar days; Weekly = one completion per Mon–Sun week; Custom = must complete on each specified `DayOfWeek`.

**Tailwind v4 theming:** Custom tokens defined in `src/index.css` via `@theme { --color-* }` — no `tailwind.config.ts`. Use semantic names: `bg-cream`, `text-cocoa`, `border-border-warm`, `bg-bubble-own`, etc.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arushawasthi-code) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
