---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Korean-Pro-Demo** is a Spring Boot 3.4.1 application for Korean pronunciation evaluation and AI-based language learning. The application integrates with external speech processing engines (SpeechPro) to provide pronunciation assessment, text-to-speech, and language learning features.

**Main Application Class:** `com.mk.KoreanProApplication` (location: `src/main/java/com/mk/KoreanProApplication.java`)

**Technology Stack:**

**Backend:**
- Java 17
- Spring Boot 3.4.1 (Data JPA, Web, Thymeleaf, WebFlux)
- PostgreSQL with QueryDSL 5.0.0
- Lombok
- Apache POI 5.2.1 for Excel/file processing
- Log4jdbc for SQL logging

**Frontend:**
- Thymeleaf template engine (server-side rendering)
- jQuery for DOM manipulation and AJAX
- RecordRTC.js for audio recording
- AOS.js for scroll animations
- Modern CSS3 with glassmorphism design patterns
- Responsive design targeting both desktop and mobile

## Building and Running

### Gradle Commands

```bash
# Build the project
./gradlew build

# Run the application (default: dev profile, port 8080)
./gradlew bootRun

# Run with specific profile
./gradlew bootRun --args='--spring.profiles.active=demo'

# Run tests
./gradlew test

# Clean build
./gradlew clean build
```

### Application Profiles

Three profiles are configured in `src/main/resources/application.yml`:
- **dev** (default): Local development with PostgreSQL at 192.168.123.181:5432, port 8080
- **prod**: Production server at 112.220.79.218:18154, port 8080
- **demo**: Demo environment at 112.220.79.218:18154, port 8081

To switch profiles, modify `spring.profiles.active` in application.yml or use command line args.

**File Storage Paths by Profile:**
- dev: `C:\data\mzcore\`
- prod: `D:\data\mzcore\`
- demo: `C:/data/mzcore/`

### Running the Application

```bash
# Default (dev profile)
./gradlew bootRun

# Demo profile (port 8081)
./gradlew bootRun --args='--spring.profiles.active=demo'
```

Access the application at `http://localhost:8080` (dev/prod) or `http://localhost:8081` (demo).

## Architecture

This application follows **Domain-Driven Design (DDD)** with a layered architecture pattern.

### Package Structure

```
com.mk
├── api/                    # Business modules (DDD bounded contexts)
│   ├── engine/            # Speech processing engine integration
│   │   ├── application/   # Service layer + DTOs
│   │   ├── domain/        # Entities (SpKoQuestion, SpKoAnswer)
│   │   ├── infrastructure/# Repositories (JPA + QueryDSL)
│   │   └── presentation/  # REST Controllers
│   ├── file/              # File upload/management
│   └── log/               # System/Web/Login logging
├── common/                 # Shared utilities
│   ├── ApiResponse        # Standard REST response wrapper
│   ├── HttpUtil           # HTTP client for external APIs
│   ├── WebClientUtil      # WebFlux-based reactive HTTP client
│   ├── FileUtil           # File operations
│   ├── Base64ToFileConverter
│   ├── ClientInfoUtil     # Client IP and request info extraction
│   └── MapConverter       # Map/Object conversion utilities
├── config/                 # Spring configuration
│   ├── exception/         # Global exception handlers
│   ├── jpa/               # QueryDSL configuration
│   ├── logging/           # AOP-based logging
│   ├── validation/        # Validation groups
│   └── webmvc/            # Web MVC configuration
└── web/                    # Thymeleaf controllers
    ├── sp/                # Speech processing UI controllers
    └── HomeController
```

### Layer Responsibilities

1. **Presentation Layer** (`presentation/`)
   - REST controllers (`@RestController`)
   - Request/response handling
   - Example: `SpKoDemoRestController`

2. **Application Layer** (`application/`)
   - Service classes containing business logic
   - DTOs for data transfer
   - Transaction boundaries (`@Service`)
   - Example: `SpDemoService`

3. **Domain Layer** (`domain/`)
   - JPA entities with business rules
   - Rich domain models using Lombok `@Builder`
   - Example: `SpKoQuestion`, `SpKoAnswer`

4. **Infrastructure Layer** (`infrastructure/`)
   - Repository interfaces extending `JpaRepository`
   - Custom repository implementations with QueryDSL
   - Pattern: `XxxRepository` extends `JpaRepository<Entity, ID>` + `XxxRepositoryCustom`
   - Custom implementation: `XxxRepositoryCustomImpl` uses QueryDSL

### External API Integration

The application integrates with **SpeechPro Korean** engine (`api.speechpro_kr.url`):

**Endpoints:**
- `/gtp` - G2P (Grapheme-to-Phoneme) conversion
- `/model` - Pronunciation model generation
- `/scorejson` - Pronunciation evaluation via base64 audio

**Service:** `SpDemoService` (src/main/java/com/mk/api/engine/application/SpDemoService.java:43)
- `createModel()` - Generates pronunciation symbols and models
- `createEvaluate()` - Evaluates pronunciation from base64 audio, saves to DB and file system

## Key Features

### 1. AOP-Based System Logging


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [babaoflamp/korean-pro-demo](https://github.com/babaoflamp/korean-pro-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
