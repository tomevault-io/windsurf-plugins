---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zettelgarden is a human-centric, open-source personal knowledge management system built on zettelkasten principles. It's a full-stack application with three main services:

- **Frontend**: React/TypeScript with Vite (`zettelkasten-front/`)
- **Backend**: Go API server (`go-backend/`)  
- **Mail Service**: Python Flask SMTP service (`python-mail/`)

## Development Commands

### Frontend (zettelkasten-front/)
```bash
cd zettelkasten-front
npm start          # Start development server (Vite)
npm run build      # Build for production (TypeScript compilation + Vite build)
npm test           # Run tests with Vitest
npm run serve      # Preview production build
```

### Backend (go-backend/)
```bash
cd go-backend
go run main.go     # Start development server
source .env-bash && go test ./...      # Run all tests
go build -o main   # Build binary
```

### Full Stack Development
```bash
# Build and deploy all services
./build.sh         # Builds Docker images and deploys via SSH

# Local development with Docker
docker-compose up  # Start all services locally
```

## Architecture

### Backend (Go)
- **Main**: `main.go` - HTTP server setup with JWT middleware, CORS, and route definitions
- **Handlers**: `handlers/` - HTTP route handlers organized by feature (auth, cards, tasks, files, etc.)
- **Models**: `models/` - Database models and business logic
- **Server**: `server/` - Database connections and server configuration
- **Migrations**: `schema/` - SQL migration files for database schema
- **LLMs**: `llms/` - AI/ML integration for embeddings, chat, and entity processing
- **Telegram**: `telegram/` - Telegram bot for chat via Telegram

### Frontend (React/TypeScript)
- **Pages**: `src/pages/` - Main application routes and page components
- **Components**: `src/components/` - Reusable UI components organized by feature
- **Contexts**: `src/contexts/` - React context providers for state management
- **API**: `src/api/` - HTTP client functions for backend communication
- **Models**: `src/models/` - TypeScript type definitions

### Key Features
- **Cards**: Atomic notes with markdown support, backlinking, starring, and AI-powered summaries/analysis
  - Card hierarchies with parent-child relationships
  - Multiple view modes: normal, summary, and analysis views
  - Linked entities and references tracking
  - Tabbed interface for files, facts, and metadata
- **Tasks**: Task management with recurring capability, priorities, and scheduling
  - Today's task counter in sidebar
  - Task creation shortcuts and dialogs
  - Task tagging and filtering
- **Files**: File upload/storage with S3 integration and card attachment
- **RSS Feed Client**: Subscribe to RSS/Atom feeds with auto-tagging support
  - Feeds: Browse and manage RSS/Atom feed subscriptions
  - Articles: Reader-style inbox for fetched articles
  - Conversion: Selectively convert interesting articles to cards
  - Folders: Organize feeds into folders for better navigation
  - Scheduled Fetch: Background job fetches new articles every 60 minutes
  - Starring: Star/unstar articles for later reference
    - Star icon in article list and reader view
    - Dedicated Starred feed in sidebar
    - Filtered API endpoint for starred articles
- **Search**: Vector search with embeddings, traditional text search, and starred searches
  - Quick search functionality with keyboard shortcuts
  - Search result starring and management
- **Entities**: Named entity recognition, management, and linking (PRO feature)
  - Entity dialogs for viewing and editing
  - Entity-card relationship tracking
- **Facts**: Structured fact management and storage (PRO feature)
- **Memory**: Personal knowledge retention and recall system
- **Starring**: Bookmark system for both cards and searches with sidebar management
- **Templates**: Card templates with variable substitution
- **Keyboard Shortcuts**: 'c' (create card), 't' (create task), 's' (search)
- **Subscription Features**: PRO gating for advanced features like entities and facts
- **Admin**: Administrative interface for managing users, jobs, and system operations
  - Job Queue monitoring and management
  - Mailing list management and history
  - Scheduled Jobs Admin
    - View all registered scheduled jobs with schedules
    - Monitor job execution status and history
    - Per-job statistics (success rate, recent runs)
    - Expandable history with pagination
    - Manual refresh for current status
  - User management and details

### Database
- PostgreSQL with pgvector extension
- Typesense as a search cache with built in embeddings
- Migration-based schema management in `go-backend/schema/`
- Models use database/sql with manual query construction

### Authentication & Authorization
- JWT-based authentication with middleware in `main.go`
- Admin-only routes protected by admin middleware
- User context passed through request context

### AI/ML Integration
- OpenAI-compatible LLM client for chat and embeddings
- Vector search available through Typesense
- Entity extraction and processing pipeline

### Testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zettelgarden/Zettelgarden](https://github.com/Zettelgarden/Zettelgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
