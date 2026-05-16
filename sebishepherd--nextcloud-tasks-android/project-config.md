---
trigger: always_on
description: **Last Updated**: 2026-02-27
---

# AI Agent Guide - Nextcloud Tasks Android

**Last Updated**: 2026-02-27
**Project Version**: 1.0.0
**Target Audience**: Claude Code, AI assistants, and automated agents working on this codebase

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Key Technologies](#key-technologies)
4. [Code Structure](#code-structure)
5. [Development Workflows](#development-workflows)
6. [Code Conventions](#code-conventions)
7. [Important Patterns](#important-patterns)
8. [Build & Deployment](#build--deployment)
9. [Testing](#testing)
10. [AI Assistant Guidelines](#ai-assistant-guidelines)
11. [Common Tasks](#common-tasks)

---

## Project Overview

**Nextcloud Tasks Android** is a native Android client for Nextcloud Tasks, built with modern Android development practices and Clean Architecture principles.

### Key Information

- **Package Name**: `com.nextcloud.tasks`
- **Language**: Kotlin
- **Min SDK**: 26 (Android 8.0)
- **Target/Compile SDK**: 36 (Android 15)
- **Java Version**: 17
- **Module Count**: 3 (app, data, domain)
- **Total Kotlin Files**: ~67

### Features

- Multi-account support with account switching
- CalDAV-based task synchronization
- VTODO (iCalendar) parsing and generation
- Offline-first architecture with Room database and pending operations queue
- Periodic background synchronization via WorkManager (every 15 minutes)
- Field-level merge conflict resolution (preserves local changes to different fields)
- Adaptive layouts for tablets and large screens (permanent navigation drawer, constrained content width)
- Pull-to-refresh synchronization
- Task lists with color coding
- Task filtering and sorting
- Network connectivity monitoring with validation grace period
- Material 3 design with dynamic theming

---

## Architecture

### Clean Architecture (3-Layer)

The project follows Clean Architecture with strict dependency rules:

```
┌─────────────────────────────────────────────────┐
│ :app (Android Application)                     │
│ • UI (Jetpack Compose)                         │
│ • ViewModels                                   │
│ • Navigation                                   │
│ • Dependency Injection Setup                   │
└─────────────────────────────────────────────────┘
                    ↓ depends on
┌─────────────────────────────────────────────────┐
│ :data (Android Library)                        │
│ • Repository Implementations                   │
│ • Room Database                                │
│ • CalDAV Service                               │
│ • Network Layer (Retrofit + OkHttp)            │
│ • iCal4j Integration (VTODO parsing/generation)│
│ • Authentication & Token Management            │
└─────────────────────────────────────────────────┘
                    ↓ depends on
┌─────────────────────────────────────────────────┐
│ :domain (Pure Kotlin Module)                   │
│ • Business Models (Task, TaskList, Tag, etc.)  │
│ • Repository Interfaces                        │
│ • Use Cases                                    │
│ • NO Android dependencies                      │
└─────────────────────────────────────────────────┘
```

### Module Details

#### `:app` Module
- **Path**: `/app`
- **Package**: `com.nextcloud.tasks`
- **Source**: `app/src/main/java/` (uses 'java' directory for Kotlin)
- **Purpose**: UI layer with Jetpack Compose screens
- **Key Files**:
  - `MainActivity.kt` - Entry point with all UI composables (adaptive layout support)
  - `TasksApp.kt` - Application class with `@HiltAndroidApp`
  - `TaskListViewModel.kt` - Task list state management and business logic
  - `auth/LoginScreen.kt`, `auth/LoginViewModel.kt` - Authentication UI
  - `sync/SyncScheduler.kt` - Periodic background sync scheduling via WorkManager
  - `sync/SyncWorker.kt` - Background sync worker implementation
  - `di/AppModule.kt` - Hilt module for use cases
  - `ui/theme/Theme.kt`, `Color.kt`, `Type.kt` - Material 3 theming

#### `:data` Module
- **Path**: `/data`
- **Package**: `com.nextcloud.tasks.data`
- **Source**: `data/src/main/kotlin/`
- **Purpose**: Data layer with repositories, database, and network
- **Key Subdirectories**:
  - `api/` - Retrofit API interfaces and DTOs
  - `caldav/` - CalDAV service, parsers (VTodoParser, DavMultistatusParser), generators (VTodoGenerator)
  - `database/` - Room database, DAOs, entities
  - `repository/` - Repository implementations (DefaultTasksRepository, DefaultAuthRepository)
  - `network/` - OkHttp clients, interceptors, SafeDns, NetworkMonitor
  - `sync/` - SyncManager, TaskFieldMerger (field-level merge), PendingOperationsManager
  - `auth/` - Authentication token provider
  - `mapper/` - Entity ↔ Domain model mappers
  - `di/` - Hilt modules (DataModule, NetworkModule)

#### `:domain` Module
- **Path**: `/domain`
- **Package**: `com.nextcloud.tasks.domain`
- **Source**: `domain/src/main/kotlin/`
- **Purpose**: Pure business logic (platform-agnostic)
- **Key Subdirectories**:
  - `model/` - Domain models (Task, TaskList, Tag, NextcloudAccount, etc.)
  - `repository/` - Repository interfaces (TasksRepository, AuthRepository)
  - `usecase/` - Use cases (LoadTasksUseCase, LoginWithPasswordUseCase, etc.)

---

## Key Technologies

### Core Stack

| Technology | Version | Purpose |
|------------|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SebiShepherd/nextcloud_tasks_android](https://github.com/SebiShepherd/nextcloud_tasks_android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
