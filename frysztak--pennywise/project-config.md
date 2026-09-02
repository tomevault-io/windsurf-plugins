---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pennywise is a full-stack expense tracking and splitting application built with Go and React. It uses Connect RPC (gRPC-like protocol) for API communication between the backend and frontend.

**Key Features:**
- Multi-currency expense tracking and splitting with weighted shares
- Money transfer recording between group members
- Real-time balance calculations integrating expenses and transfers
- Group activity feed combining all financial transactions
- Database-backed session tokens with HTTP-only cookies

**Tech Stack:**
- Backend: Go 1.25+, Connect RPC, SQLite
- Frontend: React 19, TypeScript, Vite, TanStack Router, TanStack Query, Tailwind CSS v4
- Database: SQLite with sqlc for type-safe queries
- API: Protocol Buffers (protobuf) with Connect RPC
- Logging: Structured logging with slog and tint (colored output)

## Development Commands

### Full Stack Development
```bash
# Start both frontend and backend in parallel (recommended)
just dev

# Or individually:
just web  # Start Vite dev server (localhost:5173)
just api  # Start Go backend with hot reload (localhost:3333)
```

### Backend (Go)

**Build & Run:**
```bash
go run main.go           # Production mode
go run main.go -dev      # Development mode (uses Vite dev server)
go tool air -- -dev      # With hot reload (used by `just api`)
```

**Testing:**
```bash
go test ./...                    # Run all tests
go test ./calc -v                # Run tests in specific package
go test -run TestFunctionName    # Run specific test
```

**Code Generation:**
```bash
go generate              # Runs both sqlc and buf generate
sqlc generate            # Generate database code from queries
buf generate             # Generate protobuf/Connect code
```

### Frontend (Web)

From the `web/` directory:
```bash
npm run dev              # Start dev server
npm run build            # Build for production (runs tsc then vite build)
npm run lint             # Run ESLint
npm run buf:generate     # Generate protobuf client code
npx tsc -b --noEmit      # Type check (must use -b flag, plain `tsc --noEmit` produces no output)
```

### All Code Generation
```bash
just gen  # Runs go generate + npm run buf:generate
```

## Architecture

### Backend Structure

**Entry Point:** `main.go` handles server initialization, Vite integration, and HTML template serving.

**HTTP Router:** `http/router/routes.go` registers all Connect RPC service handlers with:
- Session middleware for authentication
- Logging interceptor for request tracking
- Validation interceptor using buf.validate

**Services:** Service implementations are in `http/routes/{service}/`:
- `auth/` - Authentication (login, registration)
- `user/` - User management (profile, settings)
- `admin/` - Admin operations (placeholder)
- `group/` - Group management (create, edit, delete, member weights, activity)
- `expense/` - Expense tracking (CRUD operations)
- `transfer/` - Money transfers between group members (CRUD operations)

**Middleware & Interceptors:**
- `http/middleware/session.go` - Session authentication via database lookup for all endpoints except login/register
- `log/middleware.go` - Request logging with unique request IDs, user tracking, duration, and error codes

**Logging Infrastructure:**
- `log/logger.go` - Global slog-based logger with configurable level and format
- `log/context.go` - Context-aware logging (use `log.FromContext(ctx)` in handlers)
- Supports colored text output (via tint) or JSON format
- All requests automatically logged with request ID, procedure, user ID, duration, and error codes

**Database Layer:**
- `db/db.go` - Database initialization and migration runner
- `db/schema/` - SQL migration files (Goose migrations)
- `db/queries/` - SQL queries (sqlc format with named @parameters)
- `db/database/` - Generated sqlc code (type-safe database queries)

**Business Logic:**
- `calc/balance.go` - Group balance calculations with:
  - Weighted expense splitting per currency
  - Transfer processing (sender balance increases, receiver decreases)
  - Multi-currency support with separate balances per currency

**Helpers:**
- `http/helpers/auth.go` - GetSessionInfo() for extracting user from context
- `http/helpers/cookies.go` - SetConnectCookie() and ClearConnectCookie() for session management

**Configuration:** `config/config.go` loads environment variables from `.env` file:
- `DB_PATH` - SQLite database file path (default: "pennywise.db")
- `AUTH_SECRET` - Secret key for authentication (required)
- `LOG_LEVEL` - Logging level: debug, info, warn, error (default: "info")
- `LOG_FORMAT` - Log format: text (colored), json (default: "text")
- OIDC settings (optional, partially implemented)

### Frontend Structure

Located in `web/`:
- `src/` - React application source
- Entry: HTML template generated in `main.go` (see `indexTmpl` at main.go:184) with Vite integration
- Uses TanStack Router with file-based routing and auto code splitting
- Uses TanStack Query with Connect Query for type-safe API calls
- Shadcn/ui components built on Radix UI primitives
- Tailwind CSS v4 for styling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frysztak/pennywise](https://github.com/frysztak/pennywise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
