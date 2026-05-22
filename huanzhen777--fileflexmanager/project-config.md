---
trigger: always_on
description: **FileFlexManager** is a web-based file manager designed for Linux NAS systems with a mobile-first approach.
---

# FileFlexManager - Project Context

## Project Overview

**FileFlexManager** is a web-based file manager designed for Linux NAS systems with a mobile-first approach.

### Key Features
- File operations (browse, upload, download, delete, move, copy)
- Web-based user interface optimized for mobile devices
- Multi-user support with permission control
- Docker deployment for Linux environments

---

## Technology Stack

### Backend
- **Java**: JDK 21
- **Framework**: Spring Boot 3
- **Build Tool**: Gradle (multi-module project)
- **ORM**: MyBatis Plus
- **Object Mapping**: MapStruct
- **Database**: H2 / SQLite (embedded)
- **Architecture**: Domain-Driven Design (DDD)
- **Deployment**: Docker

### Frontend
- **Framework**: Vue 3 with Composition API
- **Language**: TypeScript
- **UI Library**: Vant (mobile-first components)
- **API Integration**: Unified configuration via `config.ts`

---

## Project Structure

### Backend Modules

```
backend/
├── domain/                    # Domain layer (core business logic)
│   ├── event/                # Domain events
│   ├── model/                # Domain models
│   ├── repository/           # Repository interfaces
│   ├── service/              # Domain services
│   └── utils/                # Domain utilities
│
├── application/              # Application layer (use cases)
│   ├── assembler/           # Application assemblers
│   ├── config/              # Application configuration
│   ├── dto/                 # Data Transfer Objects
│   ├── event/               # Application events
│   ├── scheduler/           # Scheduled tasks
│   └── service/             # Application services
│
├── infrastructure/           # Infrastructure layer
│   ├── config/              # Infrastructure configuration
│   ├── external/            # External service integrations
│   ├── persistence/         # Data persistence
│   │   ├── converter/       # Data converters
│   │   ├── entity/          # JPA entities
│   │   ├── mapper/          # MyBatis mappers
│   │   ├── po/              # Persistent objects
│   │   └── repository/      # Repository implementations
│   ├── security/            # Security implementations
│   ├── util/                # Infrastructure utilities
│   └── task/                # Task handlers
│       └── handler/
│
└── interfaces/              # Interface layer (API endpoints)
    └── src/main/
        ├── java/com.huanzhen.fileflexmanager.interfaces/
        │   ├── api/
        │   │   ├── advice/         # Global exception handlers
        │   │   ├── assembler/      # API assemblers
        │   │   └── controller/     # REST controllers
        │   ├── config/             # Interface configuration
        │   ├── controller/         # Legacy controllers
        │   ├── convert/            # Converters
        │   ├── converter/          # Additional converters
        │   ├── exception/          # Custom exceptions
        │   ├── facade/             # Facade patterns
        │   └── model/
        │       ├── dto/            # DTOs
        │       ├── req/            # Request models
        │       ├── resp/           # Response models
        │       └── vo/             # View objects
        └── resources/
            ├── data/               # Static data
            ├── db.migration/       # Database migrations
            ├── static/             # Static resources
            └── application*.yml    # Configuration files
```

### Frontend Structure

```
frontend/
├── src/
│   ├── api/              # API service layer
│   ├── assets/           # Static assets
│   ├── components/       # Reusable Vue components
│   ├── router/           # Vue Router configuration
│   ├── stores/           # Pinia stores (state management)
│   ├── utils/            # Utility functions
│   └── views/            # Page components
├── public/               # Public static files
├── dist/                 # Build output
└── config/
    ├── env/              # Environment configurations
    ├── typescript/       # TypeScript configurations
    └── build/            # Build configurations
```

---

## Architecture Principles

### 1. Domain-Driven Design (DDD)

**Layer Dependencies:**
```
interfaces → application → domain ← infrastructure
```

**Rules:**
- ✅ Strict layer boundaries - each layer only depends on the domain layer and the layer directly below it
- ✅ Domain logic isolation - business logic stays in the domain layer
- ✅ Domain models are technology-agnostic
- ❌ Never let infrastructure concerns leak into domain layer

### 2. Object Mapping with MapStruct

**Mapping Strategy:**
- **Interface Layer**: `DTO ↔ Domain Model` (using converters in `interfaces/converter/`)
- **Infrastructure Layer**: `Domain Model ↔ PO/Entity` (using converters in `infrastructure/persistence/converter/`)

**Rules:**
- ✅ Always use MapStruct for object conversions
- ✅ Define mapping interfaces with `@Mapper` annotation
- ✅ Use compile-time generation (no reflection)
- ✅ Add custom mapping rules when needed
- ❌ Never manually map objects in service layer

### 3. Data Model Layers

- **DTO (Data Transfer Object)**: Used in API layer for client communication
- **Domain Model**: Core business entities in domain layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huanzhen777/FileFlexManager](https://github.com/huanzhen777/FileFlexManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
