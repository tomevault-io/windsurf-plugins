---
trigger: always_on
description: handleSubmit = (event) => {
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

Memcode is an open-source flashcards application built with React frontend and Node.js/Express backend. It uses PostgreSQL for persistence and supports spaced repetition learning. The project also includes a secondary application called **Meresei** - a calendar app for non-24 sleep wake disorder - that runs on the same infrastructure using virtual host routing.

## Development Commands

### Database Setup
```bash
# Create/reset development database
make db-reset

# Drop database
make db-drop

# Run specific migration
make db-migrate

# Dump database for backup
make db-dump

# Restore from dump
make db-restore
```

### Development Server
```bash
# Start backend development server with auto-restart and debugging
make start

# Start frontend esbuild compilation in watch mode
make frontend-webpack

# Both commands should be run simultaneously for full development
```

### Testing
```bash
# Run backend tests
make test-backend

# Run frontend tests
make test-frontend

# Reset test database
make test-db-reset
```

**Note on Testing**: This project has minimal tests by design. Only a few critical utility functions have tests. Do not write new tests unless specifically requested - the project prioritizes rapid development and manual testing over comprehensive test coverage.

### Production/Heroku Commands
```bash
# Deploy to Heroku
make heroku-deploy

# Build for production (automatically run on Heroku)
make heroku-postbuild

# Access Heroku database console
make heroku-db-console

# Pull production database to local
make heroku-db-pull
```

## Architecture Overview

### Dual Application Setup
The codebase hosts two applications using virtual host routing:
- **Memcode** (main flashcards app): Default routes and memcode.com
- **Meresei** (calendar for non-24 sleep wake disorder): Routed via vhost for meresei.com domains

Both applications are deployed on the same Heroku server but serve completely different purposes and user bases.

### Backend Architecture (Node.js/Express)
- **Entry Point**: `backend/index.js` - Sets up vhost routing and starts server
- **API Layer**: Dynamic controller routing via `/api/:controllerName.:methodName`
- **Database**: PostgreSQL with Knex (migrating from pg-promise)
- **Models**: Located in `backend/models/` with dedicated model classes
- **Path Aliases**: Uses `#~/` for backend imports (configured in package.json)

Key backend concepts:
- Controllers follow naming pattern (e.g., `CourseApi`, `UserApi`)
- Middleware pipeline: SSL redirect → CORS → authentication → error handling
- Spaced repetition algorithm for flashcard scheduling
- **Database Migration**: Currently transitioning from pg-promise to Knex query builder

### API Response Pattern
All backend endpoints should use standardized response methods:
```javascript
response.success(obj)      // 200 status with data object
response.error(string)     // 500 status with error message  
response.validation(array) // 400 status with validation errors
```

These are injected by the `injectResponseTypes` middleware.

### Frontend Architecture (React/Redux)
- **Entry Point**: `frontend/index.js` - React app with Redux store
- **Build System**: Webpack with separate dev/production configs
- **State Management**: Redux with connected components
- **Routing**: React Router for SPA navigation
- **Path Aliases**: Uses `~/` for frontend imports

Key frontend concepts:
- Page components in `frontend/pages/` correspond to routes
- Reusable components in `frontend/components/` and `frontend/appComponents/`
- API calls through centralized services in `frontend/api/`
- CSS modules and SCSS for styling

### CSS and Styling
The project uses CSS modules with SCSS for component styling. Follow these patterns:

#### CSS Modules Pattern
```scss
// Single :local() wrapper per component
:local(.componentName) {
  .childClass {
    // styles
  }
  
  .anotherChild {
    // nested styles
    
    .deeplyNested {
      // deep nesting is preferred
      
      &:hover {
        // pseudo-selectors
      }
    }
  }
}
```

#### Component Usage
```jsx
import css from './index.scss';

// Only the main wrapper uses css object
<div className={css.componentName}>
  {/* All nested elements use string classNames */}
  <div className="childClass">
    <div className="anotherChild">
      <div className="deeplyNested">Content</div>
    </div>
  </div>
</div>
```

#### Styling Guidelines
- **Single :local() wrapper** - Only one per component file
- **Deep nesting preferred** - Unlike some CSS traditions, we embrace deep nesting for component isolation
- **String classNames** - Use string literals for nested classes, not `css.className`
- **SCSS features** - Use variables, mixins, and nesting as needed
- **Component-scoped** - All styles are automatically scoped to prevent conflicts

### Database Schema
Core entities:
- **Users**: OAuth-based authentication (GitHub/Google)
- **Courses**: Flashcard collections with categories
- **Problems**: Individual flashcards with different types
- **Learning Progress**: Spaced repetition tracking with easiness factors
- **Notifications**: User activity notifications

### Database Query Migration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lakesare/memcode](https://github.com/lakesare/memcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
