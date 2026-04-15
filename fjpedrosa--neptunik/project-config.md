---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎯 Principios Fundamentales

1. Lee archivos completos antes de modificar
2. Implementa funcionalidad real (no dummy)
3. Aplica clean architecture
4. Aplica principios SOLID
5. Los ficheros se nombran con kebab-case
6. Selecciona agentes automáticamente según contexto
7. Evalúa siempre qué agentes de los que dispones pueden ayudarte a cumplir lo que se te pide

## Behavior

1. When defining an action plan, I want that plan to also include the necessary tasks, covering both research and execution. Each task should be assigned to the appropriate expert agent or agents, and prioritized so that tasks which can be carried out in parallel are executed simultaneously to save time.

## Documentation

- **business and product** documents are located in /product/ folder, including the
  knowledger base, business and product decisions, research, and executive
  summaries
  - **development and usage documentation** is located in /docs/ folder

## Project Overview

**Neptunik** - Platform for WhatsApp Cloud API integration with complete onboarding flow.

## Technology Stack

### Core Framework & Language

- **Framework**: Next.js 15.5.2 with App Router
- **Language**: TypeScript 5.9.2
- **Runtime**: Node.js >=18.0.0

### Frontend Stack

- **Styling**: Tailwind CSS 3.4.17
- **State Management**: Redux Toolkit 2.8.2 + React Redux 9.2.0
- **UI Components**: Custom components based on Radix UI primitives
- **Forms**: React Hook Form 7.62.0 with Zod 4.1.5 validation
- **Animation**: Framer Motion 12.23.12
- **Icons**: Lucide React
- **Utilities**: class-variance-authority, tailwind-merge, clsx

### Testing & Quality Assurance

- **Unit Testing**: Jest with Next.js integration
- **Component Testing**: React Testing Library + Jest DOM
- **E2E Testing**: Playwright (Chrome, Firefox, Safari, mobile)
- **Accessibility**: WCAG 2.1 AA compliance testing
- **Coverage**: 80-95% thresholds (Domain: 95%, Application: 90%)

### Media & Export Capabilities

- **GIF Generation**: gif.js + html2canvas
- **Canvas Mocking**: jest-canvas-mock for testing
- **Web Workers**: Background GIF processing

### Analytics & Monitoring

- **Error Tracking**: @sentry/nextjs + @sentry/replay
- **Analytics**: @vercel/analytics + posthog-js
- **Performance**: @vercel/speed-insights + web-vitals 5.1
- **A/B Testing**: Custom implementation with PostHog

### Development Tools

- **Package Manager**: npm (>=8.0.0)
- **Linting**: ESLint with Next.js integration
- **Dev Server**: Turbopack (with Webpack fallback)

## Commands

### Development

```bash
# Recommended for daily use
npm run dev:stable      # Optimized stable development (8GB, GC tuned)
npm run dev:performance # High performance for complex features (12GB, aggressive GC)
npm run dev:memory      # Memory debugging mode (16GB, expose-gc)

# Specialized modes
npm run dev:hot-reload  # Optimized HMR with Fast Refresh
npm run dev:production  # Development with production-like config
npm run dev:webpack     # Webpack fallback when Turbopack fails
npm run dev:fallback    # Minimal config for limited resources

# Debug & Analysis
npm run dev:debug       # Debug mode with detailed Turbopack logging
npm run dev:clean       # Clean cache + stable development
npm run turbo:analyze   # Turbopack performance analysis

# Production
npm run build          # Build for production (8GB memory)
npm run build:analyze  # Build with bundle size analysis
npm run start          # Start production server
npm run start:turbo    # Production server with Turbopack
```

### Quick Development Scripts
```bash
# Use the shorthand script for common commands
./.claude/commands/dev              # Start stable development
./.claude/commands/dev sim          # Simulator-optimized mode
./.claude/commands/dev clean        # Clean cache and start
./.claude/commands/dev help         # Show all available commands

# Advanced utilities
./.claude/commands/dev-utils.sh smart    # Auto-select best mode based on system
./.claude/commands/dev-utils.sh memory   # Check memory usage
./.claude/commands/dev-utils.sh health   # Complete system health check
```

### Testing

```bash
# Unit & Integration Testing
npm run test          # Run Jest tests
npm run test:watch    # Jest in watch mode
npm run test:coverage # Generate coverage report
npm run test:ui       # Jest with verbose output
npm run test:component # Test UI components only
npm run test:domain   # Test domain layer only
npm run test:integration # Integration tests only

# End-to-End Testing
npm run test:e2e      # Run Playwright tests
npm run test:e2e:ui   # Playwright with UI
npm run test:e2e:debug # Debug Playwright tests

# All Tests
npm run test:all      # Run all tests (Jest + Playwright)
```

### Development Tools

```bash
# Cache Management
npm run clean:cache   # Clear Next.js and build cache
npm run clean:all     # Full reset with dependency reinstall

# Analysis
npm run turbo:analyze # Analyze Turbopack performance
npm run lint         # ESLint with TypeScript
```

## Project Structure

```
src/
├── app/                       # Next.js App Router pages
│   ├── (marketing)/          # Marketing landing pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fjpedrosa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
