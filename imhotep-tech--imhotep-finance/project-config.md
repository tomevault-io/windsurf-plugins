---
trigger: always_on
description: This file contains essential information to help AI assistants work effectively in this repository.
---

# Copilot Instructions for Imhotep Finance

This file contains essential information to help AI assistants work effectively in this repository.

## Project Overview

**Imhotep Finance** is a full-stack personal finance management platform with:
- **Backend**: Django 5.2 + Django REST Framework with multiple feature apps
- **Frontend**: React 19 + Vite + Tailwind CSS (web) + React Native + Expo (mobile)
- **Database**: PostgreSQL
- **Auth**: JWT + Google OAuth
- **API Docs**: Swagger/OpenAPI via `drf-spectacular`

The architecture separates concerns into feature-based Django apps (accounts, transaction_management, finance_management, scheduled_trans_management, target_management, user_reports, wishlist_management).

## Build & Development

### Starting the Full Stack

```bash
# Build and start all services (backend, frontend, PostgreSQL)
docker compose up --build

# Access points:
# - Frontend: http://localhost:3000
# - Backend API: http://localhost:8010
# - Swagger UI: http://localhost:8010/swagger/
# - Django Admin: http://localhost:8010/admin/
```

### Frontend Development

```bash
cd frontend/imhotep_finance

# Dev server with hot reload
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Backend Development

```bash
cd backend/imhotep_finance

# Run development server (local)
python manage.py runserver 8010

# Apply database migrations
python manage.py migrate

# Create superuser for Django Admin
python manage.py createsuperuser
```

### Mobile App Development

```bash
cd frontend/imhotep_finance_mobile

# Start Expo development server
npx expo start

# Options after startup:
# - Press 'a' for Android emulator
# - Press 'i' for iOS simulator
# - Scan QR with Expo Go app for real device testing
```

## Testing

### Backend Tests (Django)

```bash
# Run all tests
python manage.py test

# Run specific app (e.g., accounts, transaction_management)
python manage.py test accounts

# Run specific test file
python manage.py test accounts.tests.test_apis

# Run specific test class
python manage.py test accounts.tests.test_apis.UserLoginApiTest

# Run specific test method
python manage.py test accounts.tests.test_apis.UserLoginApiTest.test_login_success

# With Docker
docker exec imhotep_finance-backend-1 python manage.py test accounts --verbosity=2

# Useful flags:
# --verbosity=2 : Show each test as it runs
# --keepdb : Keep test database for debugging
# --parallel : Run tests in parallel (faster)
```

Test structure in each app:
```
app_name/
├── tests/
│   ├── test_apis.py       # API endpoint tests
│   ├── test_serializers.py  # Serializer validation tests
│   └── test_services.py   # Business logic tests
```

Test coverage: ~240 tests across all apps, covering happy path, validation, security, and edge cases.

### Frontend Tests

```bash
cd frontend/imhotep_finance
npm test
```

## Project Structure

### Backend Apps

- **`accounts/`**: User authentication, profiles, Google OAuth integration
- **`finance_management/`**: Core finance models (NetWorth, Categories), currency utilities
- **`transaction_management/`**: Transaction CRUD, CSV import, filtering
- **`scheduled_trans_management/`**: Recurring transaction automation
- **`target_management/`**: Savings goals and targets
- **`user_reports/`**: Financial reports and analytics generation
- **`wishlist_management/`**: Wishlist items and tracking

### Frontend

**Web** (`frontend/imhotep_finance/src/`):
- `components/` - Reusable React components (common, PWA)
- `pages/` - Page-level components (auth, main, profile)
- `contexts/` - React Context providers (AuthContext, ThemeContext)
- `hooks/` - Custom React hooks
- `config/` - API client setup (`config/api.js`)
- `utils/` - Utility functions

**Mobile** (`frontend/imhotep_finance_mobile/`):
- `app/` - File-based routing with Expo Router
  - `(auth)/` - Authentication screens
  - `(tabs)/` - Main app tabs (dashboard, transactions, reports, etc.)
- `components/` - Reusable React Native components
- `constants/` - API configuration, colors, types
- `contexts/` - State management (AuthContext)
- `hooks/` - Custom hooks for theme, layout
- `widgets/` - Android home-screen widgets

## Key Conventions

### Backend (Django)

1. **Architecture Pattern**: Each app is feature-based with:
   - `models.py` - Data models
   - `apis.py` - API views/viewsets (DRF)
   - `serializers.py` - Request/response serialization
   - `services.py` - Business logic (not views)
   - `selectors.py` - Query optimization (select_related, prefetch_related)
   - `tests/` - Test suite (test_apis, test_serializers, test_services)

2. **Authentication**: JWT via `rest_framework_simplejwt`
   - Access tokens: 60 minutes
   - Refresh tokens: 30 days
   - Token blacklist on rotation enabled

3. **API Documentation**: Swagger/OpenAPI via `drf-spectacular`
   - Auto-generated at `/swagger/`
   - Use docstrings on viewsets/serializers for documentation

4. **Security**:
   - All endpoints require authentication (default permission: `IsAuthenticated`)
   - User isolation: Users can only access their own resources
   - Rate limiting via throttle classes (1000 req/hr for users, 100 for anonymous)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Imhotep-Tech/imhotep_finance](https://github.com/Imhotep-Tech/imhotep_finance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
