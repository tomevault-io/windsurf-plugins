---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Build Commands
- `make build` - Build production binary with frontend assets bundled
- `make dev` - Build and run development server with DEV_MODE=true
- `make watch` - Run development server with file watching (requires air and esbuild)

### Dependencies
For watch mode development, install:
```bash
go install github.com/air-verse/air@latest
go install github.com/evanw/esbuild/cmd/esbuild@latest
```

### Go Build Tags
All Go commands must include the `--tags "fts5"` flag for SQLite FTS5 support.

## Architecture Overview

### Backend (Go)
- **Entry Point**: `main.go` - Sets up HTTP server, routes, and background tasks
- **Database**: SQLite with FTS5 for full-text search, migrations in `./migrations/`
- **Feature-based Structure**: Each feature has its own directory under `features/`
  - `notes/` - Core note management (CRUD, archive, trash, pin/unpin)
  - `tags/` - Tag management and organization
  - `focus/` - Focus modes for filtered views
  - `search/` - Full-text search with BM25 ranking
  - `images/` - Image upload and management
  - `users/` - Authentication and user management
  - `settings/` - Import/export functionality
  - `templates/` - Template management with usage tracking and placeholders
  - `intelligence/` - AI-powered indexing and similarity search (optional, requires zen-intelligence)
  - `mcp/` - Model Context Protocol server for external integrations
  - `canvas/` - Spatial organization with infinite canvas
- **Commons**: Shared utilities in `commons/`
  - `auth/` - Authentication middleware
  - `session/` - Session management
  - `sqlite/` - Database connection and migrations
  - `utils/` - HTTP utilities
  - `queue/` - Generic task queue for background processing
  - `preferences/` - User preferences (theme, view, search history)
  - `contexts/` - Preact context providers (AppContext, NotesContext)

### Frontend (Preact)
- **Entry Point**: `index.js` - Main app initialization and routing
- **Component Structure**: JSX components using Preact (React-like)
- **Build System**: esbuild bundles JSX to `assets/bundle.js`
- **Routing**: Custom router implementation in `commons/components/Router.jsx`
- **State Management**: Local component state and hooks
- **Styling**: Plain CSS with CSS custom properties for theming

### Canvas
- **Canvas Library**: Konva.js for 2D canvas rendering
- **File Format**: JSON Canvas (jsoncanvas.org) for data storage
- **Storage**: SessionStorage for canvas state persistence
- **Node Types**: Note nodes, sticky notes, image nodes
- **Features**: Pan/zoom viewport, selection, transformation, spatial organization

### Key Patterns
- **API Routes**: RESTful endpoints prefixed with `/api/`
- **Authentication**: Session-based with middleware wrapping private routes
- **File Structure**: Features are self-contained with models, handlers, and components
- **Asset Handling**: Static assets embedded in binary for production, file system for development

### Database Schema
- Migrations are sequential SQL files in `./migrations/` with format `<version>_<title>.sql`
- Uses SQLite with FTS5 extension for full-text search
- Main entities: users, notes, tags, focus_modes, sessions, images, templates, mcp_tokens, queues

### Environment Variables
- `DEV_MODE=true` - Development mode with file system assets
- `PORT` - Server port (default: 8080)
- `IMAGES_FOLDER` - Image storage path (default: ./images)
- `INTELLIGENCE_ENABLED=true` - Enable optional AI features

### Background Tasks
- Trash cleanup (30 days)
- Session cleanup (24 hours)
- Image sync from disk (24 hours)
- Intelligence queue processing (5 minutes, requires zen-intelligence)

## Code Style Guidelines

### General
- Keep code simple and readable
- Use descriptive variable names and consistent formatting
- Avoid complex language features unless necessary
- Prefer standard libraries over external dependencies
- Don't add unnecessary comments

### Preact/React Components
- Extract logic from components into separate if-else conditions
- Extract map loops to variables outside JSX
- For conditional rendering, use if-else conditions outside JSX to build arrays/variables, not ternary operators or logical AND inside JSX
- Ternary operators in JSX are only acceptable for simple inline styles or class names
- Use `is` prefix for boolean props (`isActive`, `isLoading`)
- Use `has`, `can`, `should` prefixes for other boolean checks
- Refer to `index.css` for styling patterns

## Development Conventions

### Go Backend Patterns

#### Error Handling
- Use error wrapping with context: `fmt.Errorf("context: %w", err)`
- Always log errors with `slog.Error()` before returning
- Handle `sql.ErrNoRows` separately using `errors.Is()`
- Use `panic()` only for critical initialization errors

#### HTTP Handlers
- Function signature: `func HandleXXX(w http.ResponseWriter, r *http.Request)`
- Naming: `Handle{Action}{Resource}` (e.g., `HandleGetNotes`, `HandleCreateUser`)
- Structure: Parse/validate → Business logic → Response
- Use `utils.SendErrorResponse()` for consistent error responses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sheshbabu/zen](https://github.com/sheshbabu/zen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
