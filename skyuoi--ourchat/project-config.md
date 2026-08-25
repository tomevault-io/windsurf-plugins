---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, opencode, etc.) when working with the **web-panel** portion of this repository.
---

# AGENTS.md - Web Panel

This file provides guidance to AI coding agents (Claude Code, opencode, etc.) when working with the **web-panel** portion of this repository.

## Overview

The web-panel is a Vue 3-based administration interface for managing the OurChat Rust server. It provides real-time monitoring, user management, configuration, and system logs through a modern web interface. The panel communicates with the Rust backend via gRPC-Web protocol.

## Quick Reference

### Most Common Commands

```bash
# Install dependencies
npm install

# Development server with hot reload
npm run dev

# Production build
npm run build

# Type checking
npm run type-check

# Unit tests
npm run test:unit

# End-to-end tests
npm run test:e2e

# Linting and formatting
npm run lint
npm run format
```

### Critical Notes
- Uses **Vue 3** with Composition API, **TypeScript**, and **Vite**
- **gRPC-Web** for communication with Rust backend (proxy via `/backend`)
- **Element Plus** UI component library with **Tailwind CSS**
- Base path `/panel` for reverse proxy configuration
- See sections below for detailed guidance

## Technology Stack

### Core Frameworks
- **Vue 3** (v3.5.22) with Composition API (`<script setup>` syntax)
- **TypeScript** (~5.8.3) for type safety
- **Vite** (v6.4.1) as build tool and dev server
- **Pinia** (v3.0.3) for state management
- **Vue Router** (v4.6.3) for routing
- **Vue I18n** (v11.1.10) for internationalization

### UI Components & Styling
- **Element Plus** (v2.11.7) UI component library
- **Tailwind CSS** (v4.1.16) via `@tailwindcss/vite` plugin
- **Element Plus Icons** for iconography
- **ECharts** (v5.5.1+) for data visualization
- **vue-echarts** (v7.0.3+) Vue 3 wrapper for ECharts

### gRPC Integration
- **@protobuf-ts/runtime** (v2.11.1) - Protobuf TypeScript runtime
- **@protobuf-ts/grpcweb-transport** (v2.11.1) - gRPC-Web transport
- **@improbable-eng/grpc-web** (v0.15.0) - gRPC-Web client

### Testing
- **Vitest** (v3.2.4) for unit testing
- **Playwright** (v1.56.1) for end-to-end testing

### Code Quality
- **ESLint** (v9.39.0) with Vue/TypeScript configurations
- **Prettier** (v3.5.3) for code formatting
- **Vue TSC** (v2.2.12) for type checking

## Project Structure

```
server/web-panel/
├── src/
│   ├── api/                    # Generated gRPC client code (from protobuf)
│   │   ├── google/            # Google protobuf definitions
│   │   │   ├── timestamp.ts   # Timestamp type for time-based APIs
│   │   │   └── duration.ts    # Duration type for interval-based APIs
│   │   └── service/           # OurChat service definitions
│   │       ├── auth/          # Authentication services
│   │       ├── basic/         # Basic services (server, support, etc.)
│   │       ├── ourchat/       # OurChat core services
│   │       └── server_manage/ # Server management services
│   │           └── monitoring/  # Metrics monitoring APIs
│   │               └── v1/monitoring.ts  # MonitoringMetrics types
│   ├── assets/                # Static assets
│   ├── components/            # Vue components
│   │   ├── icons/            # Icon components
│   │   ├── ConfigManager.vue
│   │   ├── Header.vue
│   │   ├── ResourceMonitor.vue
│   │   ├── Sidebar.vue
│   │   └── SystemOverview.vue
│   ├── locales/               # Internationalization files
│   │   ├── en.json
│   │   └── zh.json
│   ├── router/               # Vue Router configuration
│   │   └── index.ts
│   ├── stores/               # Pinia stores
│   │   └── grpc.ts          # gRPC connection store
│   ├── views/               # Route views/pages
│   │   ├── ConfigView.vue
│   │   ├── DashboardView.vue
│   │   ├── LoginView.vue
│   │   ├── LogsView.vue
│   │   ├── MonitorView.vue
│   │   ├── ServicesView.vue
│   │   └── UsersView.vue
│   ├── App.vue              # Root component
│   ├── main.ts             # Application entry point
│   └── shims-vue.d.ts      # TypeScript declarations
├── public/                  # Public assets
│   └── favicon.png
├── dist/                    # Build output directory
├── e2e/                    # End-to-end tests
│   ├── vue.spec.ts
│   └── tsconfig.json
└── Configuration files:
    ├── package.json
    ├── vite.config.ts
    ├── tsconfig.json
    ├── tsconfig.app.json
    ├── tsconfig.node.json
    ├── tsconfig.vitest.json
    ├── playwright.config.ts
    ├── vitest.config.ts
    ├── eslint.config.ts
    ├── .prettierrc.json
    └── .gitignore
```

## Setup and Development

### Prerequisites
- Node.js (version compatible with package.json)
- npm or pnpm (package manager)

### Installation
```bash
cd /home/limuy/OurChat/server/web-panel
npm install  # or pnpm install
```

### Development Server
```bash
npm run dev
```
- Starts Vite dev server on `http://localhost:5173/panel`
- Hot module replacement enabled
- Proxy configuration routes `/backend` to `http://localhost:7777` (Rust server)

### Type Checking
```bash
npm run type-check  # Uses vue-tsc for .vue file type checking
```

## Integration with Rust Server

### Communication Protocol
The web-panel uses **gRPC-Web** to communicate with the Rust backend. The protocol is configured via:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SkyUOI/OurChat](https://github.com/SkyUOI/OurChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
