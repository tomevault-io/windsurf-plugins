---
trigger: always_on
description: Always follow these instructions first and fallback to additional search and context gathering only if the information provided here is incomplete or found to be in error.
---

# WA-DP (Developer Portfolio) Instructions

Always follow these instructions first and fallback to additional search and context gathering only if the information provided here is incomplete or found to be in error.

## Project Overview

WA-DP is a React/TypeScript developer portfolio application with a Next.js backend API. The application allows users to create, customize, and manage their developer portfolios through a web interface.

**Architecture:**

- **Frontend**: React 19.1.0 + TypeScript + Vite (development server on port 5173)
- **Backend**: Next.js 15.3.3 API (development server on port 3000)
- **Testing**: Vitest (unit tests), Cypress (E2E tests)
- **UI Framework**: HeroUI components + Tailwind CSS
- **Build Tool**: Vite with TypeScript compilation

## Working Effectively

### Bootstrap and Install Dependencies

```bash
# Frontend dependencies (includes concurrently, testing tools, etc.)
npm ci  # Takes ~10s, may take up to 2 minutes. NEVER CANCEL.

# Backend dependencies
cd backend && npm ci  # Takes ~7s

# OR use the combined installer (requires frontend deps to be installed first)
npm run install-dev  # Installs both frontend and backend concurrently
```

**IMPORTANT**: Cypress binary download often fails due to network restrictions. If `npm ci` fails with Cypress download errors, use:

```bash
CYPRESS_INSTALL_BINARY=0 npm ci  # Skip Cypress binary, install other deps
```

### Build and Test

```bash
# Build the application
npm run build  # Takes ~14s. NEVER CANCEL. Set timeout to 60+ seconds.

# Run unit tests (424 tests across 24 files)
npm run test     # Takes ~22s. NEVER CANCEL. Set timeout to 60+ seconds.
npm run test:cov # Takes ~24s with coverage report (>94% coverage). NEVER CANCEL.

# Linting and formatting
npm run lint   # Takes ~3s. Must pass or CI will fail.
npm run format # Takes ~2s. Formats all code with Prettier.
```

### Development Servers

```bash
# Start both frontend and backend servers concurrently
npm run dev  # Frontend: http://localhost:5173, Backend: http://localhost:3000

# Start individually if needed
npm run dev:frontend  # Frontend only (port 5173)
npm run dev:backend   # Backend only (port 3000)
```

**Startup Time**: ~2s for servers to be ready. Both must be running for full functionality.

### E2E Testing

```bash
# Cypress E2E tests (requires both servers running)
npm run e2e:open  # Opens Cypress Test Runner GUI
npm run e2e:run   # Runs E2E tests headlessly (takes ~1.5 minutes)
```

**CRITICAL**: Cypress binary often fails to download due to network restrictions. E2E tests require manual installation:

```bash
npx cypress install  # May fail with "ENOTFOUND download.cypress.io"
```

If Cypress fails to install, document this limitation but proceed with other testing.

## Validation Scenarios

**ALWAYS** test these scenarios after making changes:

### Basic Application Functionality

1. **Start Development Servers**:

   ```bash
   npm run dev
   # Wait for both servers to show "ready" status
   ```

2. **Verify Frontend**: Visit http://localhost:5173
   - Portfolio displays with default user "John Doe"
   - Skills section shows progress bars
   - Work Experience and Education sections are visible
   - GitHub repositories section (may show "Failed to load" - this is normal in sandboxed environment)

3. **Verify Backend API**: Test API endpoint

   ```bash
   curl -s http://localhost:3000/api/portfolio
   # Should return: {"error":"Method not allowed"} (correct for GET request)
   ```

4. **Test User Authentication**:
   - Click "Login" button
   - Modal appears with "Create Admin Account" form
   - Enter username: `admin`, password: `password123`
   - Click "Create" - should redirect to `/edit` page
   - Portfolio editor loads with tabs: Basic Information, Social Links, Skills, Work Experience, Education

### Build and CI Validation

**ALWAYS** run these commands before committing:

```bash
npm run build     # Must succeed
npm run lint      # Must pass with no errors
npm run test:cov  # Must pass all 424 tests with >80% coverage
```

## CI/CD Pipeline Timing Expectations

The GitHub Actions pipeline has these timing expectations:

| Job                   | Expected Time | Timeout | Critical Notes                     |
| --------------------- | ------------- | ------- | ---------------------------------- |
| Lint Codebase         | ~34s          | 5 min   | Must pass or PR fails              |
| Check Formatting      | ~1m 7s        | 10 min  | Must pass or PR fails              |
| Unit Tests & Coverage | ~54s          | 15 min  | Must maintain >80% coverage        |
| E2E Tests             | ~1m 28s       | 20 min  | May fail if Cypress install issues |
| Lighthouse Report     | ~1m 44s       | 15 min  | Must score >80% in all categories  |
| Build and Deploy      | ~1m 47s       | 20 min  | Only runs on main branch           |
| SonarCloud Analysis   | ~56s          | 10 min  | Requires SONAR_TOKEN               |

**NEVER CANCEL** any of these operations. Build times can vary significantly in CI environment.

## Project Structure

```
src/                           # Frontend React application

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phillipc0/WA-DP](https://github.com/phillipc0/WA-DP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
