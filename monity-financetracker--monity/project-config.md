---
trigger: always_on
description: Monity is a full-stack personal finance management application with AI-powered transaction categorization, collaborative expense splitting, and financial health insights. Built with React and Node.js following MVC architecture.
---

# Monity Web Application - Cursor Rules

## Project Overview
Monity is a full-stack personal finance management application with AI-powered transaction categorization, collaborative expense splitting, and financial health insights. Built with React and Node.js following MVC architecture.

## Tech Stack

### Backend
- **Runtime**: Node.js 18+
- **Framework**: Express.js with MVC architecture
- **Database**: PostgreSQL via Supabase
- **Authentication**: JWT with Supabase Auth
- **AI/ML**: Custom Naive Bayes classifier with NLP (natural, compromise, stopword, ml-naivebayes)
- **Payment Processing**: Stripe API
- **Caching**: LRU cache (in-memory)
- **Scheduling**: node-cron for automated tasks
- **Security**: helmet, rate limiting (express-rate-limit), encryption middleware, input validation (joi), XSS protection
- **Testing**: Jest + Supertest

### Frontend
- **Framework**: React 19
- **Build Tool**: Vite 6
- **Styling**: Tailwind CSS 4
- **State Management**: React Context + TanStack React Query v5
- **Routing**: React Router v7
- **Charts**: react-chartjs-2
- **i18n**: react-i18next (English & Portuguese)
- **Performance**: react-window for virtualization, lazy loading
- **Analytics**: Vercel Analytics & Speed Insights
- **Testing**: Vitest + Testing Library

### Infrastructure
- **Database & Auth**: Supabase
- **API**: RESTful with versioning (/api/v1)
- **CI/CD**: AWS deployment ready

## Project Structure

### Backend Structure
```
backend/
├── server.js                      # Application entry point
├── config/                        # Environment & database config
│   ├── supabase.js               # Supabase client setup
│   └── stripe.js                 # Stripe configuration
├── models/                        # Data models (User, Transaction, Category, etc.)
├── controllers/                   # Request handlers
│   ├── authController.js
│   ├── transactionController.js
│   ├── categoryController.js
│   ├── groupController.js
│   ├── savingsGoalController.js
│   ├── aiController.js
│   ├── adminController.js
│   └── index.js                  # Controller initialization
├── services/                      # Business logic layer
│   ├── smartCategorizationService.js  # AI categorization
│   ├── aiSchedulerService.js         # Scheduled transactions
│   ├── expenseSplittingService.js    # Group expense logic
│   ├── financialHealthService.js     # Financial scoring
│   ├── scheduledTransactionService.js
│   └── index.js
├── routes/                        # API endpoint definitions
│   ├── auth.js
│   ├── transactions.js
│   ├── categories.js
│   ├── groups.js
│   ├── savingsGoals.js
│   └── index.js                  # Route aggregation
├── middleware/                    # Cross-cutting concerns
│   ├── auth.js                   # JWT authentication
│   ├── validation.js             # Input validation
│   ├── encryption.js             # Data encryption
│   ├── errorHandler.js           # Error handling
│   ├── rateLimiter.js            # Rate limiting
│   └── index.js
├── utils/                         # Helper functions
│   ├── logger.js                 # Winston logger
│   ├── helpers.js                # Utility functions
│   └── constants.js              # App constants
├── migrations/                    # Database migrations
└── __tests__/                    # Backend test suite
```

### Frontend Structure
```
frontend/src/
├── App.jsx                       # Main router & layout
├── components/                   # React components
│   ├── auth/                    # Login, Signup, etc.
│   ├── dashboard/               # Dashboard views
│   ├── transactions/            # Transaction management
│   ├── groups/                  # Expense splitting
│   ├── settings/                # Settings & preferences
│   ├── admin/                   # Admin dashboard
│   ├── ai/                      # AI features
│   ├── cashFlow/                # Cash flow calendar
│   ├── ui/                      # Reusable UI components
│   └── index.js                 # Component exports
├── context/                      # Global state
│   └── AuthContext.jsx          # Auth state management
├── hooks/                        # Custom React hooks
│   ├── usePageTracking.js       # Analytics
│   └── useLazyComponentPreloader.js
├── utils/                        # Utilities
│   ├── api.js                   # API client
│   ├── supabase.js              # Supabase client
│   └── i18n/                    # Translations (en, pt)
└── styles/                       # Global styles
```

## Architecture Guidelines

### MVC Pattern
Follow strict separation of concerns:
- **Models**: Handle data structure, validation, and database operations
- **Controllers**: Process HTTP requests, coordinate models/services, format responses
- **Routes**: Define API endpoints, apply middleware chains
- **Services**: Contain complex business logic and algorithms
- **Middleware**: Handle authentication, validation, logging, errors

### Controllers
- Accept Supabase client via dependency injection
- Use `asyncHandler` wrapper for async methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Monity-FinanceTracker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
