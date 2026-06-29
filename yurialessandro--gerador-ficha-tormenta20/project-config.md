---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gerador de Fichas de Tormenta 20 - A character sheet generator for the Brazilian tabletop RPG "Tormenta 20". The app generates random or customized character sheets with comprehensive RPG rules implementation.

### Project Structure

- **Frontend (public)**: Main React application for character sheet generation (this repository)
- **Backend (private submodule)**: Node.js backend for premium features and user authentication (located in `/backend`)
- **Premium Features (private submodule)**: Premium frontend features (located in `/src/premium`)

## Git Submodules

This project uses git submodules for backend and premium features:

- **Backend**: `git@github.com:YuriAlessandro/fichas-de-nimb-backend.git` (located in `/backend`)
- **Premium**: `git@github.com:YuriAlessandro/fichas-de-nimb-premium.git` (located in `/src/premium`)

**IMPORTANT**: New features should be developed in the premium submodule (`/src/premium`) as they will be paid features. The main repository remains open-source with core functionality.

### Working with Submodules

```bash
# Clone repository with submodules
git clone --recurse-submodules git@github.com:YuriAlessandro/gerador-ficha-tormenta20.git

# If already cloned, initialize and update submodules
git submodule init
git submodule update

# Pull latest changes from backend submodule
cd backend
git pull origin main
cd ..
git add backend
git commit -m "Update backend submodule"

# Pull latest changes from premium submodule
cd src/premium
git pull origin main
cd ../..
git add src/premium
git commit -m "Update premium submodule"

# Working in the backend
cd backend
# Make changes, commit, push as normal
git add .
git commit -m "Your commit message"
git push origin main
# Then update parent repository
cd ..
git add backend
git commit -m "Update backend submodule reference"

# Working in premium features
cd src/premium
# Make changes, commit, push as normal
git add .
git commit -m "Your commit message"
git push origin main
# Then update parent repository
cd ../..
git add src/premium
git commit -m "Update premium submodule reference"
```

## Commands

### Frontend Development

```bash
npm install          # Install frontend dependencies
npm start           # Start Vite dev server at localhost:5173
npm run build       # Build for production
```

Frontend é deployado automaticamente via Cloud Build trigger `frontend-deploy` (em `southamerica-east1`) a cada push em `main`. Config em `cloudbuild-frontend.yaml`.

### Backend Development

```bash
cd backend
npm install         # Install backend dependencies
npm run dev        # Start backend dev server
npm run build      # Build backend for production
npm start          # Start production server
```

### Full Stack Development

```bash
# Terminal 1 - Frontend
npm start

# Terminal 2 - Backend
cd backend && npm run dev
```

### Code Quality

```bash
npx tsc --noEmit   # Run TypeScript compiler check
npx eslint <filename>  # Run ESLint on specific files
npx prettier --write <filename>  # Format files with Prettier
npx prettier --check <filename>  # Check if files are formatted
```

## Architecture

### Core Structure

- **Feature-based components**: Components organized by feature (SheetResult/, DatabaseTables/)
- **Redux Toolkit**: State management with `sheetBuilder` and `sheetStorage` slices
- **TypeScript-first**: Comprehensive interfaces in `/interfaces` for all RPG entities
- **Data-driven**: All game content in `/data` folder as TypeScript objects

### Key Patterns

1. **Multi-step Character Builder**: Wizard-style form with steps for race, class, origin, attributes, equipment
2. **PDF Generation**: Uses pdf-lib to fill template PDF (`public/sheet.pdf`) with character data
3. **Persistent Storage**: Redux Persist saves character sheets locally
4. **Theme System**: Material-UI v5 with light/dark mode support

### Important Files

**Frontend:**

- `src/store/` - Redux store configuration and slices
- `src/interfaces/` - All TypeScript type definitions for RPG entities
- `src/data/` - Game content (races, classes, spells, equipment)
- `src/functions/` - Business logic and utility functions
- `src/premium/` - Premium features submodule (PRIVATE)

**Backend (when present):**

- `backend/src/` - Backend source code
- `backend/src/api/` - API routes and controllers
- `backend/src/models/` - Database models
- `backend/src/services/` - Business logic services

**Premium (when present):**

- `src/premium/` - Premium frontend features (PRIVATE)
- New paid features should be developed here
- Integrates with main app through exports

### Deprecated - DO NOT USE

- **SheetBuilder folder**: Components in `src/components/SheetBuilder/` are deprecated and should be ignored
- **t20-sheet-builder library**: This NPM package is deprecated - never import or use anything from it

### Technology Stack

- React 17 + TypeScript (strict mode)
- Vite 4.3.9 (build tool)
- Material-UI v5 + Tailwind CSS
- Redux Toolkit with Redux Persist
- pdf-lib for PDF generation

### Development Notes

- All content is in Portuguese (Brazilian)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuriAlessandro/gerador-ficha-tormenta20](https://github.com/YuriAlessandro/gerador-ficha-tormenta20) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
