---
trigger: always_on
description: EcoBuddy is a full-stack energy-saving companion app designed for teens to track their energy consumption, complete challenges, and interact with a virtual pet mascot. The application features user authentication, data persistence, social features, and gamification elements.
---

# GitHub Copilot Instructions for EcoBuddy

## Project Overview

EcoBuddy is a full-stack energy-saving companion app designed for teens to track their energy consumption, complete challenges, and interact with a virtual pet mascot. The application features user authentication, data persistence, social features, and gamification elements.

## Architecture

This is a monorepo containing:
- **Frontend**: React 18 + Vite 5 application with Tailwind CSS
- **Backend**: Express.js REST API with PostgreSQL database
- **Database**: PostgreSQL with JWT authentication

```
Frontend (Vite) → Backend API (Express) → PostgreSQL Database
```

## Tech Stack

### Frontend
- **Framework**: React 18 with functional components and hooks
- **Build Tool**: Vite 5
- **Routing**: React Router DOM v6
- **Styling**: Tailwind CSS 3 (mobile-first design)
- **Charts**: Chart.js 4 with React Chart.js 2
- **State Management**: React Context API (AuthContext)

### Backend
- **Framework**: Express.js with ES modules (`"type": "module"`)
- **Database**: PostgreSQL with `pg` driver
- **Authentication**: JWT (jsonwebtoken) + bcrypt for password hashing
- **Validation**: express-validator
- **Security**: CORS, express-rate-limit
- **Environment**: dotenv for configuration

## Code Style and Conventions

### General Guidelines
- Use ES6+ syntax with ES modules (import/export)
- Prefer functional components with hooks over class components
- Use arrow functions for component definitions
- Follow mobile-first responsive design principles
- Write descriptive variable and function names

### React Components
- Place components in `/src/components/` for reusable UI elements
- Place pages in `/src/pages/` for route components
- Use `.jsx` extension for components
- Export components as default exports
- Use React hooks (useState, useEffect, useContext) appropriately

### Styling
- Use Tailwind CSS utility classes for styling
- Brand colors are defined in `tailwind.config.cjs`:
  - Primary: `brand-primary` (#10b981 - green)
  - Accent: `brand-accent` (#6ee7b7 - light green)
- Design is mobile-first with bottom navigation bar
- Keep responsive design in mind for all components

### Backend API
- Use ES module syntax (import/export)
- Place routes in `/backend/src/routes/`
- Place models in `/backend/src/models/`
- Use middleware for authentication (`/backend/src/middleware/auth.js`)
- Validate all inputs using express-validator
- Return consistent JSON responses with appropriate status codes
- Use async/await for database operations

### Database
- Use PostgreSQL connection pool from `/backend/src/config/database.js`
- Run migrations via `npm run db:migrate` in backend directory
- Always use parameterized queries to prevent SQL injection
- Handle database errors gracefully

## Development Workflow

### Frontend Development
```bash
# Install dependencies
npm install

# Start dev server (runs on http://localhost:5173)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Backend Development
```bash
cd backend

# Install dependencies
npm install

# Set up environment variables (see .env.example)
# Create PostgreSQL database: createdb energyteen

# Run migrations
npm run db:migrate

# Start dev server with watch mode (runs on http://localhost:3001)
npm run dev

# Start production server
npm start
```

### Environment Variables

**Frontend** (`.env` in root):
```env
VITE_API_URL=http://localhost:3001/api
```

**Backend** (`.env` in `/backend/`):
```env
DATABASE_URL=postgresql://username:password@localhost:5432/energyteen
JWT_SECRET=your-secret-key-here
NODE_ENV=development
PORT=3001
FRONTEND_URL=http://localhost:5173
```

## Key Features and Components

### Authentication System
- User signup/login with JWT tokens
- AuthContext provides authentication state across the app
- Protected routes require authentication
- Tokens stored in localStorage

### Pages
- **Home** (`/`): EcoBuddy mascot with customization, seeds, streaks, levels
- **Analytics** (`/analytics`): Energy usage tracking with charts
- **Leaderboard** (`/leaderboard`): Friends rankings and social features
- **Tasks** (`/tasks`): Challenges that users can complete for rewards
- **Profile** (`/profile`): User profile and settings
- **Login/Signup**: Authentication pages

### Currency System
- **Seeds**: Primary currency earned by completing tasks
- Users can feed/play with EcoBuddy using seeds
- Seeds shown on leaderboard rankings

### Data Visualization
- Use Chart.js for all charts (line charts, pie/doughnut charts)
- Components: `ChartLine.jsx`, `ChartPie.jsx`, `KpiCard.jsx`
- Show real data only, zeros for new users

## Testing and Quality Assurance

### Before Committing
1. Test the feature locally in dev mode
2. Test both frontend and backend if changes affect API
3. Verify mobile responsiveness (the app is mobile-first)
4. Check console for errors or warnings
5. Ensure no sensitive data (API keys, passwords) are committed

### Manual Testing
- Test authentication flow (signup, login, logout)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zting052) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
