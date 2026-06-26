---
trigger: always_on
description: <frontend_aesthetics>
---

<frontend_aesthetics>
You tend to converge toward generic, flat dashboard designs. For this task, you must avoid the "corporate SaaS" look. Instead, target a **Cinematic, Immersive Media Aesthetic**. The goal is a high-contrast, deep-dark interface that feels sleek, modern, and entertainment-focused.

Focus on:

- **Color & Theme:** Commit to an **OLED-friendly dark mode**. Use absolute black (`#000000`) or deep charcoal (`#0F0F0F`) for backgrounds to make content pop. Use **Neon Accents** for high-priority actions: Electric Green (like `#2ECC71` or neon lime) for toggles/progress and Hot Red (like `#FF4757`) for primary "Subscribe" or "Live" buttons. High contrast is non-negotiable.
- **Surfaces & Glassmorphism:** Create depth using transparency and blur rather than borders. Implement **frosted glass effects** (`backdrop-filter: blur()`) for floating elements like modals, toasts, or upload status cards. These elements should feel like they are hovering above the content with subtle, dark drop shadows.
- **Typography:** Choose clean, **Geometric Sans-Serif** fonts (like circular variants or wide grotesques) that remain legible against dark backgrounds. Use heavy weights for headings and distinct colors (light gray vs. white) to establish hierarchy between titles and metadata (views, dates).
- **Geometry & Layout:** Soften the UI with **generous border-radius**. Video containers, buttons, and floating cards should have rounded corners (12px to 24px). The layout should be spacious, allowing the media thumbnails to dominate the visual hierarchy without clutter.

Avoid generic design choices:

- Low-contrast gray backgrounds (aim for deep darks instead).
- Boxy, sharp-edged containers (default to rounded).
- Heavy use of borders (use background distinctness and spacing instead).
- Standard "Bootstrap" blue primary buttons (use unique neon accents).

Interpret creatively: This is a media consumption and creation interface. It needs to feel "premium" and "app-like." Prioritize the visual impact of the video content and the "Uploading" micro-interaction state shown in the reference.
</frontend_aesthetics>

# Project Context: GoonHub

## 1. Project Overview

**GoonHub** is a high-performance, self-hosted web application designed to organize, view, and manage local NSFW video libraries. Ideally, it functions as a "single-binary" application where the frontend is embedded into the backend.

**Core Philosophy:**

- **Privacy First:** No telemetry, local-only storage, panic buttons.
- **Performance:** Capable of handling 10,000+ files with instant search.
- **User Experience:** Netflix-like UI with advanced filtering and VR support.

---

## 2. Technology Stack

### Backend (The Host)

- **Language:** Go (Golang) 1.21+
- **Web Framework:** Gin.
- **Database:** PostgreSQL 18 using **GORM** (ORM).
- **Video Processing:** FFmpeg (via `u2takey/ffmpeg-go` wrapper).
- **Image Processing:** `disintegration/imaging` for thumbnails.
- **Filesystem:** `fsnotify` for watching folder changes.

### Frontend (The UI)

- **Framework:** Vue.js 3 with Nuxt 3 (Composition API, `<script setup>`).
- **Build Tool:** Vite (SPA mode).
- **Language:** TypeScript.
- **State Management:** Pinia.
- **Styling:** Tailwind CSS.
- **Video Player:** Plyr.

### Deployment & Distribution

- **Strategy:** "Bake" the compiled frontend `dist` folder into the Go binary using `//go:embed`.
- **Dev Mode:** Frontend runs on port 3000 (proxying API), Backend on port 8080.
- **Prod Mode:** Single binary runs on port 8080.

---

## 3. Architecture & Directory Structure

Adhere strictly to the **Clean Architecture** principles and **Standard Go Project Layout**:

```text
/
├── cmd/
│   └── server/
│       └── main.go              # Application bootstrap with wire DI
├── internal/
│   ├── api/
│   │   ├── middleware/          # Request ID, Auth, Recovery, Rate Limit
│   │   ├── v1/                  # Versioned API handlers
│   │   └── router.go            # Route registration
│   ├── config/                  # Configuration (Viper)
│   ├── core/                    # Business Logic (Services) - To be refactored to domain/app
│   ├── data/                    # Data Access - To be refactored to infrastructure/persistence
│   ├── domain/                  # Domain entities and interfaces
│   ├── infrastructure/
│   │   ├── logging/             # Structured logging (Zap)
│   │   ├── persistence/         # Database implementation
│   │   └── server/              # HTTP Server wrapper
│   ├── wire/                    # Dependency Injection wiring
│   └── pkg/                     # Internal shared utilities
├── pkg/
│   ├── ffmpeg/                  # FFmpeg helper functions
│   └── scraper/                 # Logic for ThePornDB/IAFD scraping
├── web/                         # Vue.js Source Code
│   ├── src/
│   └── dist/                    # Built assets (Gitignored, but embedded by Go)
├── docker/                      # Docker Compose for PostgreSQL
└── AGENTS.md                    # This file
```

---

## 4. Database Schema (Draft)

The AI agent should use these relationships when generating models.

- **Video:** `ID, Path, Title, Hash (MD5/pHash), Duration, Size, Rating, ViewCount, CreatedAt`
    - _HasMany_ Tags, Performers
    - _BelongsTo_ Studio

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gglafrance/GoonHub](https://github.com/gglafrance/GoonHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
