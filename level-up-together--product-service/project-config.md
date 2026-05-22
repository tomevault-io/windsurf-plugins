---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Build
./gradlew clean build

# Run ALL tests (2470 tests across 5 modules)
./gradlew test

# Run tests by module
./gradlew :service:test             # all service + global tests
./gradlew :app:test                 # application context, benchmark

# Run single test class
./gradlew :service:test --tests "*.Oauth2ControllerTest"

# Run single test method
./gradlew :service:test --tests "*.Oauth2ControllerTest.getOauth2LoginUri"

# Parallel build
./gradlew test --parallel

# Run application
./gradlew bootRun                                    # Default (port 8443)
./gradlew bootRun --args='--spring.profiles.active=test'  # Test profile (port 18080)

# Generate API documentation
./gradlew openapi3 && ./gradlew sortOpenApiJson && ./gradlew copySortedOpenApiJson

# Generate GraphQL classes from DGS schema
./gradlew generateJava

# Test coverage report (minimum 70%)
./gradlew test jacocoTestReport
# Report: app/build/reports/jacoco/html/index.html
```

## Architecture Overview

**Multi-Service Monolith**: Spring Boot 3.4.5 application organized as a Gradle multi-module project (2 modules +
composite build). Services share a single deployment unit but use separate databases per service. Designed for future
MSA migration with Saga pattern.

### Gradle Multi-Module Structure

```
product-service/
├── settings.gradle                    # 2 modules: service, app + includeBuild platform
├── build.gradle                       # Root: common settings, BOM, -parameters flag
├── service/build.gradle               # ALL 12 services + global infra (single compilation, multi-srcDirs)
│   ├── src/main/java/                 # Global infra (datasource, security, profanity, translation 등)
│   ├── shared-test/src/test/          # Shared test utils (ControllerTestConfig, MockUtil)
│   ├── user-service/src/main/java/
│   ├── guild-service/src/main/java/
│   ├── ... (12 directories)
│   └── support-service/src/main/java/
└── app/build.gradle                   # Bootstrap + DGS codegen + JaCoCo
    ├── src/main/java/                 # LevelUpTogetherMvpApplication.java
    ├── src/main/resources/            # All config files, schemas, keystores
    └── src/test/java/                 # @SpringBootTest tests only (3 files)
```

**Platform shared library** (`../level-up-together-platform`): `includeBuild`로 IDE에서 소스 편집 가능. CI에서는 GitHub Packages
Maven artifact 사용.

- `lut-platform-kernel` — 순수 공유 타입, audit entity, API result, enums
- `lut-platform-infra` — 공통 Spring infra (Redis, security, handler 등)
- `lut-platform-saga` — Saga 프레임워크 + SagaDataSourceConfig

**Why single service module**: Circular dependencies between services (user↔guild, user↔gamification, user↔support,
guild↔gamification) prevent independent Gradle modules. Directories are separated for logical boundaries, compiled as
one unit via `sourceSets.main.java.srcDirs`.

### Service Modules (`service/{name}/src/main/java/io/pinkspider/leveluptogethermvp/`)

| Service               | Database        | Purpose                                                                                                            |
|-----------------------|-----------------|--------------------------------------------------------------------------------------------------------------------|
| `userservice`         | user_db         | OAuth2 authentication (Google, Kakao, Apple), JWT tokens, profiles, friends, quests                                |
| `missionservice`      | mission_db      | Mission definition, progress tracking, Saga orchestration, mission book, daily mission instances (pinned missions) |
| `guildservice`        | guild_db        | Guild creation/management, members, experience/levels, bulletin board, territory, invitations                      |
| `chatservice`         | chat_db         | Guild chat messaging, chat participants, read status, direct messages                                              |
| `metaservice`         | meta_db         | Common codes, calendar holidays, Redis-cached metadata, level configuration, attendance reward configuration       |
| `feedservice`         | feed_db         | Activity feed (CQRS Read Model), likes, comments, feed visibility, FeedProjectionEventListener                     |
| `notificationservice` | notification_db | Push notifications, notification preferences, notification management                                              |
| `adminservice`        | admin_db        | Home banners, featured content (players, guilds, feeds)                                                            |
| `gamificationservice` | gamification_db | Titles, achievements, user stats, experience/levels, attendance tracking, events, seasons                          |
| `bffservice`          | -               | Backend-for-Frontend API aggregation, unified search                                                               |
| `noticeservice`       | -               | Notice/announcement management (layered: api, application, core, domain)                                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Level-Up-Together/product-service](https://github.com/Level-Up-Together/product-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
