---
trigger: always_on
description: 1. **DO NOT break existing functionality** - All features are critical and must continue to work
---

# Cleanuparr - Claude AI Rules

## Rules

1. **DO NOT break existing functionality** - All features are critical and must continue to work
2. **When in doubt, ASK** - Don't assume, clarify with the maintainer first
3. **Always read existing code before making changes** - Understand the current architecture and patterns
4. **Follow existing patterns** - Study the codebase style and match it exactly
5. **Ask before introducing new patterns** - Use current coding standards or get approval first
6. **Prefer editing existing files over creating new ones** - Build on existing work
7. **Flag potential gotchas or issues immediately** - Document and report anything unexpected
8. **If unsure about an approach, ask before implementing**

## Project Overview

Cleanuparr is a tool for automating the cleanup of unwanted or blocked files in Sonarr, Radarr, Lidarr, Readarr, Whisparr and supported download clients (qBittorrent, Transmission, Deluge, uTorrent, rTorrent). It provides malware protection, automated cleanup, and queue management for *arr applications.

**Key Features:**
- Strike system for bad downloads
- Malware detection and blocking
- Automatic search triggering after removal (Seeker)
- Missing and upgrade search
- Orphaned download cleanup with cross-seed support
- Authentication (OIDC, 2FA)
- Notification providers (Apprise, Discord, Gotify, Notifiarr, Ntfy, Pushover, Telegram)

## Architecture & Tech Stack

### Backend
- **.NET 10.0** (C#) with ASP.NET Core
- **Architecture**: Clean Architecture with `Features/` subdirectory pattern
  - `Cleanuparr.Api` - REST API and web host (`Features/` for endpoint groups)
  - `Cleanuparr.Application` - Application services and use cases
  - `Cleanuparr.Domain` - Domain models (Entities, Enums, Exceptions)
  - `Cleanuparr.Infrastructure` - External integrations (`Features/` for Arr, DownloadClient, Notifications, etc.)
  - `Cleanuparr.Persistence` - Data access with EF Core (SQLite)
  - `Cleanuparr.Shared` - Shared utilities
- **Database**: SQLite with Entity Framework Core
  - Three separate contexts: `DataContext`, `EventsContext`, `UsersContext`
- **Key Libraries**:
  - MassTransit (messaging)
  - Quartz.NET (scheduling)
  - Serilog (logging)
  - SignalR (real-time communication)
- **Testing**: xUnit + NSubstitute + Shouldly
  - Always use **NSubstitute** for mocking in new tests (Moq is being phased out)

### Frontend
- **Angular 21** with TypeScript 5.9 (standalone components, zoneless, OnPush)
- **UI**: Custom glassmorphism design system with 33 custom components — no external UI frameworks
- **Icons**: @ng-icons/core + @ng-icons/tabler-icons
- **Design System**: 3-layer SCSS (`_variables` -> `_tokens` -> `_themes`), dark/light themes
- **State Management**: @ngrx/signals (Angular signals-based)
- **Real-time Updates**: @microsoft/signalr 10.0.0
- **PWA**: Service Worker support enabled

## Project Structure

```
Cleanuparr/
├── code/
│   ├── backend/
│   │   ├── Cleanuparr.Api/              # REST API (Features/ for endpoint groups)
│   │   ├── Cleanuparr.Api.Tests/        # API layer tests
│   │   ├── Cleanuparr.Application/      # Business logic layer
│   │   ├── Cleanuparr.Domain/           # Domain models
│   │   ├── Cleanuparr.Infrastructure/   # External integrations (Features/ subdirs)
│   │   ├── Cleanuparr.Infrastructure.Tests/
│   │   ├── Cleanuparr.Persistence/      # SQLite data access
│   │   ├── Cleanuparr.Persistence.Tests/
│   │   └── Cleanuparr.Shared/           # Shared utilities
│   ├── frontend/                        # Angular 21 application
│   ├── e2e/                             # Playwright E2E tests
│   ├── Dockerfile                       # Multi-stage Docker build
│   ├── entrypoint.sh                    # Docker entrypoint
│   └── Makefile                         # Build & migration helpers
├── docs/                                # Docusaurus documentation
├── .github/workflows/                   # CI/CD pipelines
├── blacklist                            # Default malware patterns (strict)
├── blacklist_permissive                 # Less strict malware patterns
├── whitelist                            # Safe file extensions
└── whitelist_with_subtitles             # Includes subtitle formats
```

## Code Standards & Conventions

**IMPORTANT:** Always study existing code in the relevant area before making changes. Match the existing style exactly.

### Backend (C#)
- Follow Microsoft C# Coding Conventions
- Use nullable reference types (`<Nullable>enable</Nullable>`)
- Add XML documentation comments for public APIs
- Use meaningful names - avoid abbreviations unless widely understood
- Keep services focused - single responsibility principle
- New integrations go under `Features/` subdirectories (e.g., `Infrastructure/Features/Arr/`)

### Frontend (TypeScript/Angular)
- All components must be **standalone** with **ChangeDetectionStrategy.OnPush**
- Use `input()` / `output()` function APIs (not `@Input()` / `@Output()` decorators)
- Use Angular **signals** for reactive state (`signal()`, `computed()`, `effect()`)
- Follow the 3-layer SCSS design system (`_variables` -> `_tokens` -> `_themes`)
- **Do not introduce external UI frameworks** (no PrimeNG, Material, Tailwind, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cleanuparr/Cleanuparr](https://github.com/Cleanuparr/Cleanuparr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
