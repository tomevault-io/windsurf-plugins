---
trigger: always_on
description: PigeonPod is a podcast aggregation and subscription service with a front-end and back-end separation model.
---

# AGENTS.md

## 1. Project Overview
PigeonPod is a podcast aggregation and subscription service with a front-end and back-end separation model. 
It focuses on automatically converting YouTube channels/playlists into podcast channels, automatically updating and downloading new shows, and providing online listening and standard RSS feeds.
The front-end uses Vite + React, and the back-end runs on Spring Boot framework, combined with SQlite database.

## 2. Main Tech Stack
- front-end：React 19、Vite 7、React Router 7、Mantine 8、i18next、Axios
- back-end：Java 17、Spring Boot 3.5、MyBatis-Plus、Flyway、Sa-Token、YouTube Data API v3、Rome
- database：SQLite

## 3. Code Style & Rules
- Coding language：
  - front-end: JavaScript/JSX（not TypeScript）`
  - back-end: Java 17
- Components：React Functional Components + Hooks (For example, `function App()`, `use*` custom Hooks)
- Naming Conventions：
  - Front-end variables/functions: camelCase
  - React Components: PascalCase
  - Back-end class: PascalCase; Methods/fields: camelCase
- CSS solution: Mantine component library + global/regular `.css` files (not CSS Modules)

## 4. Project Structure
The structure below reflects the current source layout. Build artifacts (`frontend/node_modules`, `frontend/dist`, `backend/target`) are omitted.

```text
pigeon-pod/                                             // Project root
├── .github/                                            // GitHub workflow definitions and repository metadata
├── backend/                                            // Spring Boot backend (API, RSS, scheduler, download pipeline)
│   ├── pom.xml                                         // Maven dependencies and build config
│   └── src/
│       └── main/
│           ├── java/
│           │   └── top/
│           │       └── asimov/
│           │           └── pigeon/                    // Backend core package (App.java entrypoint)
│           │               ├── config/                // Spring/framework config (Async/CORS/Locale/MyBatis/Sa-Token/API key)
│           │               ├── controller/            // REST API controllers and web entry points (Account/Feed/Episode/Rss...)
│           │               ├── event/                 // Domain events (download task, episodes created)
│           │               ├── exception/             // BusinessException and global exception handler
│           │               ├── handler/               // Feed/download handling pipeline (channel & playlist handlers)
│           │               ├── helper/                // YouTube/task helper components for focused integration logic
│           │               ├── listener/              // Event listeners triggered by domain events
│           │               ├── mapper/                // MyBatis mapper interfaces for SQLite CRUD
│           │               ├── model/                 // Shared model package
│           │               │   ├── constant/          // Domain constants (for example YouTube constants)
│           │               │   ├── dto/               // Internal DTOs for service/handler orchestration
│           │               │   ├── entity/            // Persistence entities (Channel/Episode/Playlist/User...)
│           │               │   ├── enums/             // Domain enums (feed type/source, episode status, batch action...)
│           │               │   ├── request/           // API request payload models
│           │               │   └── response/          // API response/result payload models
│           │               ├── scheduler/             // Scheduled tasks (sync, queue dispatch, cleanup)
│           │               ├── service/               // Business services (auth/feed/episode/dashboard/media/rss/yt-dlp)
│           │               └── util/                  // Utility classes (password, yt-dlp args validator, etc.)
│           └── resources/
│               ├── application.yml                    // Spring Boot runtime configuration
│               ├── db/
│               │   └── migration/                    // Flyway migration scripts (V* and R* SQL)
│               ├── messages*.properties               // i18n message bundles
│               └── static/
│                   ├── index.html                    // Packaged frontend entry page served by backend
│                   └── assets/                       // Built frontend JS/CSS/SVG assets
├── frontend/                                           // Vite + React frontend project
│   ├── public/                                         // Static files copied directly in build output
│   └── src/
│       ├── assets/                                     // UI images/branding assets
│       ├── components/                                 // Reusable components (layout, header, login, modals, cards)
│       │   ├── FeedCard/                               // Feed card component and styles
│       │   ├── GlobalPlayer/                           // Global media player component
│       │   └── StatisticsCard/                         // Dashboard statistics card and styles
│       ├── constants/                                  // Frontend constants (date format/subtitle/toast)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aizhimou/pigeon-pod](https://github.com/aizhimou/pigeon-pod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
