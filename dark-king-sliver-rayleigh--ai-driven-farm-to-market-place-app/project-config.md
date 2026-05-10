---
trigger: always_on
description: > This file is read by Claude Code at the start of every conversation.
---

# CLAUDE.md — Project-Level Instructions

> This file is read by Claude Code at the start of every conversation.
> It provides project-specific context, conventions, and rules.

## Project Overview

**AgroDirect** is an AI-driven Farm-to-Consumer (F2C) marketplace that connects farmers directly with consumers, eliminating intermediaries. The platform features AI-powered price insights, crop recommendations, demand forecasting, logistics tracking, route planning, and an intelligent chatbot.

## Tech Stack

### Frontend
- **Framework:** React 18 + Vite 5
- **Routing:** React Router DOM v6
- **Styling:** TailwindCSS v3
- **Charts:** Recharts
- **Maps:** Leaflet / React-Leaflet + MapLibre GL
- **State:** React Context API (AuthContext, LogisticsTrackingContext)
- **Testing:** Vitest + jsdom

### Backend
- **Runtime:** Node.js with Express 4
- **Database:** MongoDB via Mongoose 8
- **Auth:** JWT (jsonwebtoken) + bcryptjs
- **AI:** Google Gemini AI (`@google/generative-ai`)
- **ML:** ml-regression, ml-matrix, decision-tree, simple-statistics
- **Scheduling:** node-cron

### External APIs
- **Data.gov.in** — Real-time market/mandi price data
- **Geoapify** — Map-based logistics routing
- **Google Gemini** — AI chatbot, crop recommendations, sentiment analysis

## Project Structure

```
├── src/                    # Frontend (React/Vite)
│   ├── App.jsx             # Root component with routing
│   ├── main.jsx            # Vite entry point
│   ├── components/         # Reusable UI components
│   │   ├── ui/             # Base UI primitives (StatusComponents, etc.)
│   │   ├── farmer/         # Farmer-specific components
│   │   └── integrated/     # Cross-role integrated components
│   ├── pages/              # Route-level page components
│   │   └── integrated/     # Cross-role pages
│   ├── context/            # React Context providers
│   ├── hooks/              # Custom React hooks (useApi, useData, usePriceInsight)
│   ├── services/           # Frontend API client & business logic
│   ├── store/              # Client-side state management
│   └── utils/              # Utility functions
├── server/                 # Backend (Express/Node)
│   ├── server.js           # Entry point, middleware, route registration
│   ├── config/             # DB connection config
│   ├── controllers/        # Route handlers (thin — delegate to services)
│   ├── services/           # Business logic & AI/ML services
│   ├── models/             # Mongoose schemas
│   ├── routes/             # Express route definitions
│   ├── middleware/          # Auth, error handling, logging
│   ├── utils/              # Schedulers, helpers
│   ├── data/               # Seed/static data
│   └── docs/               # API documentation (Postman collection)
├── tests/                  # Frontend tests (Vitest)
├── public/                 # Static assets
└── dist/                   # Production build output
```

## Architecture & Patterns

### Backend Architecture: Controllers → Services → Models
- **Controllers** are thin — they parse requests and call services.
- **Services** contain all business logic, AI/ML processing, and external API calls.
- **Models** are Mongoose schemas with validation. Do NOT put business logic here.

### Frontend Architecture: Pages → Components + Hooks + Services
- **Pages** are route-level components, composed of smaller components.
- **Hooks** (`useApi`, `useData`, `usePriceInsight`, `useNewFeatures`) encapsulate data fetching and side effects.
- **Services** (`api.js`, `authService.js`, `priceEngine.js`, `trackingService.js`) handle HTTP calls and complex client-side logic.
- **Context** (`AuthContext`, `LogisticsTrackingContext`) provides cross-cutting state.

### User Roles
The app has three distinct user roles with separate dashboards and permissions:
1. **FARMER** — Manages products, views price insights, demand forecasts, crop recommendations
2. **CONSUMER** — Browses products, places orders, leaves feedback
3. **LOGISTICS** — Manages deliveries, route planning, real-time tracking

Role-based access is enforced via `authorizeRoles()` middleware on the backend.

## Key Commands

```bash
# Run both frontend and backend concurrently
npm run dev

# Run frontend only (Vite dev server on port 5173+)
npm run dev:frontend

# Run backend only (Express on port 5000+)
npm run dev:backend

# Run tests
npm test

# Build for production
npm run build
```

## Coding Conventions

### General
- Use **ES modules** (`import/export`) on the frontend; **CommonJS** (`require/module.exports`) on the backend.
- Environment variables: frontend uses `.env` at root (prefixed `VITE_`), backend uses `server/.env`.
- Always check `.env.example` files for required environment variable documentation.
- Never commit `.env` files.

### JavaScript / React
- Functional components only. No class components.
- Use custom hooks for reusable logic (see `src/hooks/`).
- Prefer named exports for components, default exports for pages.
- API calls go through `src/services/api.js` — do NOT use raw `fetch` in components.

### Backend
- Follow the Controller → Service → Model pattern strictly.
- All API routes are prefixed with `/api/`.
- Use the existing `errorHandler` middleware — throw errors, don't catch and manually respond.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dark-king-sliver-rayleigh/AI-Driven-farm-to-market-place-app](https://github.com/Dark-king-sliver-rayleigh/AI-Driven-farm-to-market-place-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
