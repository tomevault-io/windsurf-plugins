---
trigger: always_on
description: This file provides guidance to Gemini Code Assist when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Gemini Code Assist when working with code in this repository.

## Table of Contents
1. [⚠️ CRITICAL: Safety First](#️-critical-safety-first)
2. [Quick Start Guide](#quick-start-guide)
3. [Development Environment](#development-environment)
4. [Build & Verification](#build--verification)
5. [Code Quality Standards](#code-quality-standards)
6. [Architecture Essentials](#architecture-essentials)
7. [Documentation Index](#documentation-index)
8. [Repository & Collaboration](#repository--collaboration)

---

# ⚠️ CRITICAL: Safety First

## Commands That WILL Break Development

### ❌ FORBIDDEN WEBADMIN COMMANDS
**These commands break the development container and force a 5+ minute restart:**
- `npm run build` (anywhere in WebAdmin directory)
- `cd WebAdmin && npm run build`
- `./scripts/dev/dev-workflow.sh build-webadmin` (production testing only)

**Why?** The development container uses an isolated `.next` directory. Running `npm run build` on the host corrupts the container's build state.

### ✅ SAFE WEBADMIN VERIFICATION
Use these commands instead to verify WebAdmin changes:
- `npm run lint` - Check ESLint errors
- `npm run type-check` - Verify TypeScript types
- Hot reloading automatically validates code changes as you save files.

### ❌ FORBIDDEN DEVELOPMENT COMMANDS
- `docker compose up` for development (always use `./scripts/dev/start-dev.sh`)

**If you run forbidden commands, you will:**
1. Break the development environment.
2. Force a restart with `--clean` (which takes 5+ minutes).
3. Waste time and ignore explicit instructions.

---

# Quick Start Guide

## Starting Development Services

**⚠️ CANONICAL DEVELOPMENT STARTUP:**
```bash
./scripts/dev/start-dev.sh
```

### Available Flags
```bash
./scripts/dev/start-dev.sh              # Standard startup
./scripts/dev/start-dev.sh --webadmin   # Rebuild WebAdmin container
./scripts/dev/start-dev.sh --clean      # Complete reset (removes all volumes)
./scripts/dev/start-dev.sh --build      # Force rebuild with --no-cache
./scripts/dev/start-dev.sh --help       # Show usage
```

**Flag Details:**
- `--webadmin`: Restarts the WebAdmin container, which is useful for fixing Next.js issues or after adding new packages.
- `--clean`: Removes containers, volumes, `node_modules`, and build artifacts for a complete reset.
- `--build`: Rebuilds containers using the `--no-cache` flag.

## Available Services
After startup, these services are available:
- 🌐 **WebAdmin**: http://localhost:3000 (Next.js with hot reloading)
- 📚 **Gateway API Swagger**: http://localhost:5000/swagger
- 🔧 **Admin API Swagger**: http://localhost:5002/swagger
- 🐰 **RabbitMQ Management**: http://localhost:15672 (user: `conduit`, pass: `conduitpass`)

## Quick Verification
```bash
docker ps                              # Check running containers
docker compose -f docker-compose.yml -f docker-compose.dev.yml logs -f [service]
```

---

# Development Environment

## How Development Works

### Key Features
- ✅ Node modules exist on the HOST, allowing direct `npm` command access.
- ✅ The `WebAdmin` directory is mounted for hot reloading.
- ✅ User ID mapping prevents permission issues by using your host UID/GID.
- ✅ Development containers use `node:22-alpine` directly.
- ✅ Isolated `.next` directories mean the container manages its own build state.

### Development vs Production

| Aspect | Development (`start-dev.sh`) | Production (`docker compose up`) |
|--------|------------------------------|----------------------------------|
| WebAdmin Container | `node:22-alpine` with mounted source | Built Next.js app in container |
| Hot Reloading | ✅ Enabled via volume mounts | ❌ Static build |
| User Permissions | Maps to host UID/GID | Runs as container user |
| Node Modules | Shared with host | Container-only |

## Helper Commands

### dev-workflow.sh
This script simplifies interaction with the development containers.
```bash
./scripts/dev/dev-workflow.sh logs                 # View WebAdmin logs in real-time
./scripts/dev/dev-workflow.sh shell                # Open a shell inside the WebAdmin container
./scripts/dev/dev-workflow.sh lint-fix-webadmin    # Run ESLint with --fix
./scripts/dev/dev-workflow.sh build-sdks           # Build all SDKs
./scripts/dev/dev-workflow.sh exec [command]       # Execute a custom command in the container
```

---

# Build & Verification

## ⚠️ CRITICAL: Always Verify Builds

### WebAdmin Verification (SAFE)
**Never use `npm run build` in development - it breaks the container!**

```bash
cd WebAdmin
npm run lint         # Check for ESLint errors
npm run type-check   # Verify TypeScript types
```

### Backend & SDK Build Commands
```bash
# Full solution build
dotnet build

# Run all tests
dotnet test

# Build individual projects
dotnet build ConduitLLM.Gateway    # Gateway API
dotnet build ConduitLLM.Admin   # Admin API

# Build SDKs
cd SDKs/Node/Admin && npm run build
cd SDKs/Node/Core && npm run build
cd SDKs/Node/Common && npm run build
```

## Incremental Development Rules

1. **NEVER** make more than 3-5 file changes without verifying.
2. **ALWAYS** verify after **ANY** changes:
   - **WebAdmin**: `npm run lint` and `npm run type-check` ONLY.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickna/Conduit](https://github.com/nickna/Conduit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
