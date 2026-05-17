---
trigger: always_on
description: -  This document provides guidelines for using Copilot effectively in the DumbAssets project.
---

# Copilot Instructions for DumbAssets Project
-  This document provides guidelines for using Copilot effectively in the DumbAssets project.
-  It covers project conventions, architecture, and best practices to follow when writing code.
-  The goal is to maintain a consistent codebase that is easy to read, understand, and maintain.
-  Copilot should assist in generating code that adheres to these conventions and patterns.

# DumbAssets Architecture & Conventions

## Project Philosophy
- Keep code simple, smart, and follow best practices
- Don't over-engineer for the sake of engineering
- Use standard conventions and patterns
- Write human-readable code
- Keep it simple so the app just works
- Follow the principle: "Make it work, make it right, make it fast"
- Comments should explain "why" behind the code in more complex functions
- Overcommented code is better than undercommented code

## Commit Conventions
- Use Conventional Commits format:
  - feat: new features
  - fix: bug fixes
  - docs: documentation changes
  - style: formatting, missing semi colons, etc.
  - refactor: code changes that neither fix bugs nor add features
  - test: adding or modifying tests
  - chore: updating build tasks, package manager configs, etc.
- Each commit should be atomic and focused
- Write clear, descriptive commit messages

## Project Structure

### Root Directory
- Keep root directory clean with only essential files
- Production configuration files in root:
  - docker-compose.yml
  - Dockerfile
  - package.json
  - README.md
  - server.js (main application server)
  - nodemon.json (development configuration)

### Backend Structure
- server.js: Main Express server with all API routes
- middleware/: Custom middleware modules
  - cors.js: CORS configuration
  - demo.js: Demo mode middleware
- data/: JSON file storage
  - Assets.json: Main asset data
  - SubAssets.json: Sub-asset data
  - Images/, Manuals/, Receipts/: File uploads

### Frontend Structure (/public)
- All client-side code in /public directory
- **Manager Pattern**: Feature-specific classes in `/public/managers/`
  - globalHandlers.js: Global utilities (toaster, error logging, API calls)
  - dashboardManager.js: Dashboard rendering and charts
  - modalManager.js: Modal operations for assets/sub-assets
  - settingsManager.js: Settings modal and configuration
  - import.js: Import functionality and file processing
  - maintenanceManager.js: Maintenance event management
  - charts.js: Chart.js wrapper and chart management
  - toaster.js: Toast notification system

### Services Architecture (/src/services)
- **fileUpload/**: Modular file upload system
  - index.js: Main export interface
  - fileUploader.js: Core upload logic
  - init.js: Easy initialization
  - utils.js: Upload utilities
  - example.js: Usage examples
- **notifications/**: Apprise notification system
  - appriseNotifier.js: Apprise CLI integration
  - notificationQueue.js: Queue management
  - warrantyCron.js: Scheduled notifications
  - utils.js: Notification utilities
- **render/**: Rendering services
  - assetRenderer.js: Asset detail rendering
  - listRenderer.js: Asset list and search
  - previewRenderer.js: File preview generation
  - syncHelper.js: State synchronization
  - index.js: Service exports

### Helper Modules (/public/helpers)
- utils.js: General utility functions (generateId, formatDate, formatCurrency)
- paths.js: Path management utilities
- serviceWorkerHelper.js: PWA service worker management

### UI Enhancement (/public/js)
- collapsible.js: Collapsible section functionality
- datepicker-enhancement.js: Enhanced date input UX

# Documentation
- Main README.md in root focuses on production deployment
- Each service module has its own README.md with usage examples
- Code must be self-documenting with clear naming
- Complex logic must include comments explaining "why" not "what"
- JSDoc comments for public functions and APIs
- File headers must explain module purpose and key functionality

# Module System & ES6
- Use ES6 modules with import/export syntax
- Each manager class should be in its own file
- Services should be modular and reusable
- Use named exports for utilities, default exports for main classes
- Import statements at the top of files
- Dynamic imports only when necessary for performance

# Manager Pattern (/public/managers)
- Each major feature has its own manager class
- Manager classes handle feature-specific logic and DOM manipulation
- Managers should not directly manipulate other managers' DOM elements
- Use dependency injection for shared utilities
- Manager constructors should accept configuration objects
- Each manager should have clear initialization and cleanup methods

# Service Architecture (/src/services)
- Services are backend utilities that can be used across the application
- Each service directory should have:
  - index.js: Main export interface
  - README.md: Documentation and examples
  - Specific implementation files
- Services should be stateless when possible
- Use consistent error handling across services

# Global Handlers Pattern
- globalHandlers.js centralizes common frontend functionality
- Exposes utilities to globalThis for app-wide access:
  - globalThis.validateResponse: API response validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DumbWareio/DumbAssets](https://github.com/DumbWareio/DumbAssets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
