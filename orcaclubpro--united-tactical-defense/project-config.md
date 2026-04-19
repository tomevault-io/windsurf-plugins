---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Frontend Development
```bash
# Start frontend development server
cd frontend && npm start

# Build frontend for production
cd frontend && npm run build:prod

# Build frontend with optimized assets
cd frontend && npm run build:optimized

# Run frontend tests
cd frontend && npm test

# Analyze bundle size
cd frontend && npm run analyze

# Optimize assets before build
cd frontend && npm run optimize-assets

# Serve production build locally
cd frontend && npm run serve:prod

# Preload critical assets
cd frontend && npm run preload-critical-assets
```

### Full Stack Development
```bash
# Install all dependencies (root + frontend)
npm run install-all

# Start both frontend and backend concurrently
npm start

# Build frontend only
npm run build
```

### Testing
```bash
# Run all tests
npm run test:all

# Run backend API tests
npm run test:api

# Run service layer tests  
npm run test:services

# Run integration tests
npm run test:integration

# Run E2E tests
npm run test:e2e

# Run frontend tests
npm run test:frontend

# Run performance tests
npm run test:performance

# Generate test coverage report
npm run test:coverage

# Generate comprehensive test report
npm run test:report
```

### Database Operations
```bash
# Run database migrations up
npm run migrate

# Run database migrations down
npm run migrate:down

# Test database connection pooling
npm run test:db-pooling
```

## Project Architecture

### Frontend Architecture (React + TypeScript)
- **Framework**: React 18 with TypeScript
- **Routing**: React Router DOM with lazy loading  
- **Styling**: SCSS with component-scoped styles + Styled Components for dynamic styles
- **State Management**: React hooks with context for global state
- **Build Tool**: Create React App with custom optimizations
- **Analytics**: Google Analytics 4 integration with custom tracking
- **Icons**: FontAwesome React components and React Icons
- **Charts**: Chart.js with react-chartjs-2 for dashboard analytics

### Backend Architecture (Hybrid: Deprecated + Serverless)
- **Current**: Serverless functions (Netlify) with mock data fallbacks in `frontend/netlify/functions/`
- **Deprecated**: Express.js server with SQLite database (in `/deprecated/`)
- **External APIs**: LeadConnector API for appointment booking
- **Database**: Transitioned from SQLite to external services
- **Development Mode**: Mock implementations with `logMockOperation` indicators in console

### Key Directories
- `frontend/src/components/`: React components organized by feature
- `frontend/src/services/`: API layer and external service integrations
- `frontend/src/utils/`: Utility functions and helper modules
- `deprecated/`: Legacy Express.js backend (reference only)
- `netlify/functions/`: Serverless API endpoints

## Code Architecture Patterns

### Component Organization
- **Landing Page**: Modular sections (Hero, Programs, Testimonials, etc.) in `frontend/src/components/landing/`
- **Dashboard**: Analytics dashboard with charts and metrics in `frontend/src/components/dashboard/`
- **Booking**: Appointment booking flow with calendar integration in `frontend/src/components/booking/`
- **Common**: Shared components (Footer, modals, etc.) in `frontend/src/components/common/`
- **Form Components**: Booking forms and modals in `frontend/src/components/Form/`

### API Layer Pattern
- Mock implementations for development (`frontend/src/services/api.ts`)
- Fallback to real endpoints when available
- Comprehensive TypeScript interfaces for all data models
- Error handling with graceful degradation

### Analytics Integration
- Custom analytics service with GA4
- Page visit tracking with user journey analysis
- Conversion tracking and funnel analysis
- Real-time metrics dashboard

## External Service Integrations

### LeadConnector API
- **Purpose**: External appointment booking system
- **Endpoint**: `https://backend.leadconnectorhq.com/appengine/appointment`
- **Format**: Multipart form data submission
- **Fields**: `first_name`, `last_name`, `email`, `phone`, `selected_slot`, `tag`

### Firebase Services
- **Analytics**: Event tracking and user behavior analysis
- **Configuration**: Environment-based initialization
- **Hosting**: Static asset delivery (if used)

### Netlify Platform
- **Hosting**: Frontend deployment with automatic builds
- **Functions**: Serverless API endpoints
- **Forms**: Contact form handling
- **Redirects**: SPA routing support

## Development Workflow

### Local Development Setup
1. Install dependencies: `npm run install-all`
2. Start development: `npm start` (runs both frontend and deprecated backend concurrently)
3. Frontend runs on `http://localhost:3000`
4. Deprecated backend API runs on `http://localhost:3001`
5. For frontend-only development: `cd frontend && npm start`

### Testing Strategy
- **Unit Tests**: Component and utility testing with Jest/React Testing Library
- **Integration Tests**: API endpoint testing with Mocha/Chai
- **Performance Tests**: Load testing with autocannon
- **E2E Tests**: Full user journey testing

### Build & Deployment
- **Development**: `npm start` for hot reload

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orcaclubpro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
