---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-service application called "ServiceNow Helper" that provides an AI-powered interface for ServiceNow assistance featuring **multi-agent AI architecture**. The entire system is containerized using Docker Compose.

The core components are:
- **Next.js 16.0.0 Frontend**: A web application for user interaction, authentication, and displaying results with agent model configuration, knowledge store management, and multimodal capabilities.
- **n8n Workflow Engine (1.118.2)**: Handles the backend AI processing with multi-agent support, integrating with services like Anthropic Claude.
- **PostgreSQL Database (15.x with pgvector 0.8.1)**: Provides data storage for n8n, session storage, agent model configurations, and knowledge store for the Next.js app.
- **ServiceNow Helper Companion App**: A ServiceNow application that facilitates integration using staging table architecture for secure data synchronization.

## Development Commands

- `docker compose up -d` - **Recommended method.** Starts all services.
- `docker compose --profile setup up -d` - **First time setup.** Starts all services and runs initial setup.
- `./scripts/setup-n8n.sh` - Manual setup script (only needed if automatic setup fails).
- `npm run dev` - Start the Next.js development server (requires separate n8n/Postgres instances).
- `npm run build` - Build production application with PWA optimization.
- `npm run build:analyze` - Build with webpack bundle analyzer for performance insights.
- `npm run start` - Start production server.
- `npm run lint` - Run ESLint checks with enhanced code quality rules.
- `npm run type-check` - Run TypeScript type checking.
- `npm test` - Run Jest unit tests (non-interactive mode).
- `npm run test:watch` - Run Jest tests in watch mode.
- `npm run test:coverage` - Run Jest tests with coverage reporting.
- `npm run test:ci` - Run Jest unit tests for CI/CD environments.
- `npm run test:e2e` - Run Playwright integration tests.
- `npm run test:e2e:ui` - Run Playwright tests with UI mode.
- `npm run test:e2e:headed` - Run Playwright tests in headed mode.
- `npm run test:e2e:debug` - Run Playwright tests in debug mode.
- `npm run test:performance` - Run performance-specific tests.

## Architecture

The application follows a containerized, multi-service architecture orchestrated by Docker Compose.

### Authentication System
- JWT-based authentication using httpOnly cookies.
- Auth middleware in `src/lib/auth-middleware.ts` verifies tokens server-side.
- `AuthContext` provides client-side authentication state management.

### API Integration & Data Flow (Multi-Agent Streaming Architecture)
1.  User submits a question via the `SearchInterface` in the Next.js app.
2.  The request is sent to `/api/submit-question-stream`, which is protected by JWT authentication and includes agent model configurations.
3.  The Next.js API establishes a **Server-Sent Events (SSE) streaming connection** to the n8n webhook with agent-specific model settings.
4.  **Real-time streaming**: Response chunks are streamed back immediately as n8n generates them using configured agent models (Orchestration, Business Rule, Client Script).
5.  The UI updates in real-time, displaying content as it streams, similar to ChatGPT/Claude interfaces.
6.  Enhanced cancellation system allows users to stop streaming requests at any time.
7.  **Script Deployment**: Users can deploy generated scripts directly to ServiceNow via `/api/send-script` endpoint, which uses the N8NClient server-side to communicate with N8N workflows.

### Settings System
- **User Settings Management**: Persistent user preferences stored in PostgreSQL database
- **Agent Model Configuration**: Individual AI model selection per specialized agent with expandable UI cards
- **AI Model Management**: Comprehensive AI model management with capabilities tracking (text, image, audio)
- **Settings Context**: React context providing settings state management and API integration
- **AgentModelContext**: Dedicated React context for multi-agent model state management
- **AIModelContext**: React context for AI model state and configuration management
- **ThemeContext**: Dark/light theme management with persistent user preferences
- **Authentication-Aware**: Settings are user-specific and require authentication to save
- **Real-time Sync**: Settings changes immediately reflect across the application
- **Default Fallbacks**: Works gracefully when unauthenticated with sensible defaults

### Multi-Agent AI Architecture
- **Orchestration Agent**: Coordinates overall response and routing between different specialized agents
- **Business Rule Agent**: Specialized for ServiceNow business logic and rule configuration
- **Client Script Agent**: Optimized for client-side scripting and UI component development
- **Scalable Design**: Architecture supports unlimited agents without code changes
- **Individual Model Configuration**: Each agent can use a different AI model optimized for its tasks
- **Database Migration**: Comprehensive migration system from single-model to multi-agent configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanielMadsenDK/servicenow-helper](https://github.com/DanielMadsenDK/servicenow-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
