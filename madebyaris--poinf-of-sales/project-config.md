---
trigger: always_on
description: This is a **modern, enterprise-grade Point of Sale system** built with cutting-edge technologies:
---


# 🍽️ POS System - Complete Architecture Guide

## 🎯 Project Overview

This is a **modern, enterprise-grade Point of Sale system** built with cutting-edge technologies:

### 🏗️ Technology Stack
- **Backend:** Golang 1.21+ + Gin framework + PostgreSQL with optimized raw SQL
- **Frontend:** React 18.3+ + TanStack Start + TypeScript 5.6+ + Tailwind CSS + shadcn/ui
- **Database:** PostgreSQL 15 with comprehensive schema, indexes, and constraints
- **Infrastructure:** Docker containers with multi-stage builds and Docker Compose orchestration
- **Authentication:** JWT-based auth with role-based access control (RBAC)

### 🚀 Core Business Features
- **Multi-Role Support:** Admin, Manager, Server, Counter, Kitchen interfaces
- **Complete Order Lifecycle:** Creation → Kitchen → Payment → Completion
- **Real-time Updates:** WebSocket-like updates across all interfaces
- **Advanced Admin Tables:** Professional data tables with sorting, filtering, pagination
- **Payment Processing:** Multi-step payment flow with receipt generation
- **Kitchen Display System:** Real-time order preparation workflow
- **Table Management:** Advanced seating and table assignment system

## 📁 Project Structure

### 🏠 Root Level Files
- [Makefile](mdc:Makefile) - **PRIMARY DEV TOOL** - All development commands (`make dev`, `make backup`, etc.)
- [docker-compose.dev.yml](mdc:docker-compose.dev.yml) - Development environment with hot reloading
- [docker-compose.yml](mdc:docker-compose.yml) - Production orchestration
- [README.md](mdc:README.md) - Complete project documentation with screenshots
- [ROLE_BASED_IMPLEMENTATION.md](mdc:ROLE_BASED_IMPLEMENTATION.md) - Role system implementation guide

### 🔧 Backend Structure (`backend/`)
- [main.go](mdc:backend/main.go) - Application entry point, CORS, middleware setup
- [go.mod](mdc:backend/go.mod) - Go 1.21+ dependencies (Gin, PostgreSQL driver, JWT)
- [Dockerfile](mdc:backend/Dockerfile) + [Dockerfile.dev](mdc:backend/Dockerfile.dev) - Multi-stage container builds
- **`internal/`** - Clean Go architecture following best practices:
  - **[api/routes.go](mdc:backend/internal/api/routes.go)** - RESTful API route definitions with role-based grouping
  - **[models/models.go](mdc:backend/internal/models/models.go)** - Complete data models, DTOs, and API response structures
  - **[database/connection.go](mdc:backend/internal/database/connection.go)** - PostgreSQL connection with pooling
  - **[middleware/auth.go](mdc:backend/internal/middleware/auth.go)** - JWT authentication + RBAC middleware
  - **`handlers/`** - Domain-specific HTTP handlers:
    - [auth.go](mdc:backend/internal/handlers/auth.go) - Login, logout, user management
    - [orders.go](mdc:backend/internal/handlers/orders.go) - Complete order lifecycle management
    - [products.go](mdc:backend/internal/handlers/products.go) - Menu and category management
    - [tables.go](mdc:backend/internal/handlers/tables.go) - Table and seating management
    - [payments.go](mdc:backend/internal/handlers/payments.go) - Payment processing and history

### ⚛️ Frontend Structure (`frontend/`)
- [package.json](mdc:frontend/package.json) - React 18.3+, TypeScript 5.6+, TanStack ecosystem
- [vite.config.ts](mdc:frontend/vite.config.ts) - Vite + TanStack Start configuration
- [tailwind.config.js](mdc:frontend/tailwind.config.js) - Tailwind CSS + shadcn/ui theme configuration
- [Dockerfile](mdc:frontend/Dockerfile) + [Dockerfile.dev](mdc:frontend/Dockerfile.dev) - Nginx + Node.js containers
- **`src/`** - Modern React application structure:
  - **[main.tsx](mdc:frontend/src/main.tsx)** - React 18 entry point with StrictMode
  - **[index.css](mdc:frontend/src/index.css)** - Global styles and CSS variables
  - **Core Architecture:**
    - **[types/index.ts](mdc:frontend/src/types/index.ts)** - Comprehensive TypeScript definitions
    - **[api/client.ts](mdc:frontend/src/api/client.ts)** - Axios-based API client with interceptors
    - **[lib/utils.ts](mdc:frontend/src/lib/utils.ts)** - Utility functions (cn, date formatters, etc.)
    - **[lib/form-schemas.ts](mdc:frontend/src/lib/form-schemas.ts)** - Zod validation schemas
  - **UI Components:**
    - **[components/ui/](mdc:frontend/src/components/ui/)** - shadcn/ui base components (Button, Card, Table, etc.)
    - **[components/forms/](mdc:frontend/src/components/forms/)** - Reusable form components with validation
  - **Business Components:**
    - **[components/admin/](mdc:frontend/src/components/admin/)** - Admin dashboard and management interfaces
    - **[components/pos/](mdc:frontend/src/components/pos/)** - POS interface components (cart, product grid, etc.)
    - **[components/kitchen/](mdc:frontend/src/components/kitchen/)** - Kitchen display system components
    - **[components/server/](mdc:frontend/src/components/server/)** - Server-specific interface components
    - **[components/counter/](mdc:frontend/src/components/counter/)** - Counter/checkout interface components
  - **Routing:**
    - **[routes/](mdc:frontend/src/routes/)** - TanStack Start file-based routing
    - **[routeTree.gen.ts](mdc:frontend/src/routeTree.gen.ts)** - Auto-generated route tree
  - **Hooks & State:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
