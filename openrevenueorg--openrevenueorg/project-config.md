---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**OpenRevenue** is an open-source alternative to TrustMRR that allows startups to verify and showcase their revenue transparently. The platform consists of two main components:

1. **OpenRevenue Platform**: The main web application for discovering and showcasing verified startups
2. **OpenRevenue Standalone App**: A self-hosted data provider that indies/startups can install on their own servers

The platform provides multiple integration options, granular privacy controls, and community-driven features for transparent revenue sharing.

## Technology Stack

### Main Platform
- **Frontend**: Next.js 14+ (TypeScript) with App Router, Tailwind CSS, Shadcn/ui
- **Backend**: Next.js API Routes (serverless), BullMQ + Redis for background jobs
- **Database**: PostgreSQL 15+ with Redis 7+ for caching
- **Authentication**: NextAuth.js v5
- **Package Manager**: pnpm
- **Runtime**: Node.js 20+

### Standalone App
- **Backend Framework**: Express.js or Fastify (Node.js)
- **Frontend**: Next.js with static export or vanilla HTML/CSS/JS
- **Database**: SQLite (for simplicity) or PostgreSQL
- **Authentication**: JWT-based API keys + session management for UI
- **Package Manager**: pnpm
- **Runtime**: Node.js 20+
- **Deployment**: Docker container for easy self-hosting
- **Data Integrity**: Cryptographic signatures for API responses

## Development Commands

### Main Platform
```bash
# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your values

# Set up database
pnpm db:push

# Seed development data
pnpm db:seed

# Start development server
pnpm dev

# Build for production
pnpm build

# Run tests
pnpm test

# Run E2E tests
pnpm test:e2e

# Linting and formatting
pnpm lint
pnpm lint:fix
pnpm format

# Type checking
pnpm typecheck

# Database operations
pnpm db:migrate
pnpm db:reset
pnpm db:studio
```

### Standalone App
```bash
# Navigate to standalone app directory
cd packages/standalone

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env
# Edit .env with your values

# Initialize database
pnpm db:init

# Start development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Build Docker image
docker build -t openrevenue-standalone .

# Run with Docker
docker run -p 3001:3001 -v ./data:/app/data openrevenue-standalone

# Access UI
# API: http://localhost:3001/api/v1
# Web UI: http://localhost:3001
```

## Architecture Overview

### High-Level Structure
```
┌─────────────────────────────────────────────────────────────┐
│                    OPENREVENUE PLATFORM                      │
├─────────────────────────────────────────────────────────────┤
│  Web App (Next.js)  │  Embeddable Widget  │  Public API     │
└──────────────┬──────────────────┬──────────────────┬────────┘
               │                  │                  │
┌──────────────▼──────────────────▼──────────────────▼────────┐
│                      APPLICATION LAYER                       │
├─────────────────────────────────────────────────────────────┤
│  API Routes  │  Auth Service  │  Data Aggregator             │
└──────────────┬─────────────────────────────────────┬────────┘
               │                                     │
┌──────────────▼─────────────────────────────────────▼────────┐
│                       DATA LAYER                             │
├─────────────────────────────────────────────────────────────┤
│  PostgreSQL  │  Redis Cache  │  Encrypted Storage (Vault)   │
└──────────────┬─────────────────────────────────────┬────────┘
               │                                     │
┌──────────────▼─────────────────────────────────────▼────────┐
│                   DATA SOURCES                               │
├─────────────────────────────────────────────────────────────┤
│  Direct APIs     │        Standalone Apps                    │
│  (Stripe,        │        ┌─────────────────┐               │
│   Paddle, etc.)  │        │ User's Server   │               │
│                  │        │ ┌─────────────┐ │               │
│                  │        │ │ Standalone  │ │               │
│                  │        │ │ App + API   │ │               │
│                  │        │ └─────────────┘ │               │
│                  │        └─────────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

### Key Components

#### Main Platform
- **Data Aggregator**: Unified service to fetch data from multiple sources (direct APIs + standalone apps)
- **Payment Providers**: Abstract interface with implementations for direct API integrations
- **Standalone App Client**: HTTP client for communicating with self-hosted standalone apps
- **Revenue Sync**: Background jobs for scheduled synchronization from all data sources
- **Leaderboard**: Materialized views with multi-layer caching (edge, Redis, database)
- **Dashboard**: User management interface with startup settings and analytics

#### Standalone App

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openrevenueorg/openrevenueorg](https://github.com/openrevenueorg/openrevenueorg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
