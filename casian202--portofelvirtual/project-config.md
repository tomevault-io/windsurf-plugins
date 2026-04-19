---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **PortofelVirtual** (Romanian for "Virtual Wallet") - a full-stack personal budget management application with:
- **Frontend**: React 18 + Vite + TailwindCSS + shadcn/ui + React Query + React Router
- **Backend Express**: Node.js + Express + PostgreSQL + WebSocket (port 3001)
- **Backend FastAPI**: Python + FastAPI + PostgreSQL (port 9000) - for AI integration
- **Authentication**: JWT-based with password change enforcement
- **Timezone**: Europe/Bucharest (Romania) across all dates

## Commands

### Frontend (root directory)
```bash
npm run dev          # Start development server (Vite on port 5173)
npm run build        # Build for production
npm run lint         # Run ESLint
npm run lint:fix     # Fix ESLint issues
npm run typecheck    # Run TypeScript type checking
```

### Backend Express (`backend/` directory)
```bash
cd backend
npm run dev          # Start with hot reload (--watch)
npm run start        # Start production server
npm run migrate      # Run migrations manually
npm run seed         # Seed database with initial data
```

### Backend FastAPI (`fastapi_app/` directory)
```bash
cd fastapi_app
pip install -r requirements.txt   # Install Python dependencies
uvicorn main:app --reload --port 9000   # Development server
# Or use the provided scripts:
./run.sh        # Linux/Mac
run.bat         # Windows
```

### Docker (full stack)
```bash
docker compose up --build    # Build and run all services
```

Services started:
- Frontend (Nginx): port 8889
- Backend Express: port 3001
- FastAPI: port 9000
- PostgreSQL: port 5433 (external)

## Architecture

### Dual Backend Architecture

This project has **two backend implementations** that share the same PostgreSQL database:

1. **Express Backend** (Node.js) - Primary API for the frontend React app
   - WebSocket support for real-time updates
   - Used by the web application

2. **FastAPI Backend** (Python) - AI-friendly API
   - OpenAPI documentation at `/docs`
   - Special `/api/ai/*` endpoints for OpenWebUI integration
   - API Key authentication support
   - Used by AI assistants and external integrations

### Frontend Structure
```
src/
├── api/apiClient.js         # Axios instance + WebSocket + all API methods
├── lib/
│   ├── AuthContext.jsx       # Auth state management (login, logout, checkAuth)
│   ├── query-client.js       # React Query client configuration
│   └── utils.js             # Utility functions (cn merge, date helpers)
├── components/
│   ├── ui/                  # shadcn/ui components (Dialog, Button, Table, etc.)
│   ├── finance/             # Transaction, Category, Charts components
│   ├── investments/         # Investment cards and forms
│   └── goals/               # Goal components
├── pages/                   # Page components (Dashboard, Incomes, Expenses, etc.)
├── pages.config.js          # Page routing configuration
├── App.jsx                  # Main app with ProtectedRoute, DashboardRoute
└── main.jsx                 # Entry point
```

### Backend Express Structure
```
backend/
├── src/
│   ├── server.js            # Express app + WebSocket + auto-migrations
│   ├── db.js               # PostgreSQL connection + query helper
│   ├── middleware/auth.js  # JWT authentication middleware
│   ├── routes/             # API routes (auth, transactions, categories, etc.)
│   └── seed.js             # Database seeding
└── migrations/             # SQL migration files (001-005)
```

### FastAPI Structure
```
fastapi_app/
├── main.py                  # FastAPI app with CORS, routers
├── database.py              # PostgreSQL connection pool
├── auth.py                  # JWT + API Key authentication
├── models.py                # Pydantic models
├── routes/
│   ├── auth.py              # Login, user management
│   ├── transactions.py      # CRUD transactions
│   ├── categories.py        # CRUD categories
│   ├── investments.py       # CRUD investments
│   ├── goals.py             # CRUD savings goals
│   ├── wallet.py            # Balance, summary, meal vouchers
│   ├── ai.py                # AI-specific endpoints
│   ├── admin.py             # User management (admin only)
│   ├── exchange_rates.py    # Currency rates
│   └── api_keys.py          # API key management
├── requirements.txt
└── Dockerfile
```

### Database Schema
- **users**: id, email, password_hash, full_name, role, must_change_password
- **transactions**: id, amount, type, category_id, category_name, description, date, month, currency, is_recurring, recurring_day, is_meal_voucher
- **budget_categories**: id, name, type, icon, color, is_active
- **investments**: id, name, type, initial_amount, current_value, purchase_date
- **savings_goals**: id, name, target_amount, current_amount, deadline, currency, is_completed
- **exchange_rates**: id, currency, rate (relative to RON)
- **api_keys**: id, user_id, name, key, is_active, created_at, last_used_at

### Key Patterns

1. **Page Configuration**: All pages are registered in `pages.config.js`:
   ```javascript
   export const pagesConfig = {
       mainPage: "Dashboard",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Casian202) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
