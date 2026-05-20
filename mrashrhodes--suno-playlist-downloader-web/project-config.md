---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands
- `yarn dev` - Start the development server
- `yarn build` - Build the application (TypeScript + Vite)
- `yarn tauri` - Run Tauri commands
- `yarn preview` - Preview the built application

## Code Style Guidelines
- **Imports**: Group imports by type (React, components, services, plugins)
- **Types**: Use explicit TypeScript interfaces and types, with strict typing when possible
- **Naming**: Use camelCase for variables/functions, PascalCase for components/classes
- **Error Handling**: Use try/catch with appropriate error messages
- **Components**: Functional components with React hooks
- **CSS**: Use Mantine components and style props, CSS modules for custom styling
- **State Management**: React useState/useEffect hooks, avoid global state when possible
- **File Structure**: Components in src/components, services in src/services
- **File Naming**: Component files use PascalCase (.tsx), service files use PascalCase (.ts)

## Version Control
- Use git to track changes and make rollback easy

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Suno Playlist Downloader — Visual Modernization**

A web-based tool that downloads music from Suno playlists and user profiles as ZIP archives with embedded ID3 metadata. It's live on Replit and fully functional. This project focuses exclusively on visual modernization — upgrading the UI to a premium dark-first design with algorithmic art, while preserving all existing functionality.

**Core Value:** The app must continue to work exactly as it does now — every download flow, every setting, every API call unchanged. Visual changes only.

### Constraints

- **No functional changes**: Every download flow, API call, and setting must continue working identically
- **Mantine v6**: Cannot upgrade — too many breaking changes, risk to functionality
- **Replit deployment**: Must remain deployable on Replit with current build process
- **Build process**: `build.sh` and Vite config must continue to work
- **Client-only changes**: All modifications confined to `client/src/`
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- JavaScript (Node.js) - Server-side backend, ES modules with `type: "module"`
- TypeScript 5.0.4 - Client-side frontend with React components
- HTML/CSS - Client UI styling with Mantine framework
- JSDoc - Documentation in service files where TypeScript not used
## Runtime
- Node.js 20 (minimum 16.0.0) - Server runtime on Replit and production
- Browser (ES2020+) - Client runtime, React 18
- npm (root level for server and client dependencies)
- Lockfile: `package-lock.json` present
## Frameworks
- Express.js 4.19.2 - HTTP server framework
- React 18.2.0 - Client-side UI framework
- Vite 4.3.9 - Client build tool and dev server
- Mantine 6.0.13 - React component library with hooks
- @tabler/icons-react 2.20.0 - Icon library
- Vite 4.3.9 - React build and dev server
- @vitejs/plugin-react 3.1.0 - React Fast Refresh plugin
- PostCSS 8.4.24 - CSS processing
## Key Dependencies
- puppeteer 24.9.0 - Headless browser automation for Suno profile scraping with infinite scroll
- node-fetch 3.3.2 - HTTP client for API requests (ESM compatible)
- express-session 1.18.0 - Session management for user preferences storage
- adm-zip 0.5.10 - ZIP file creation for batch downloads
- node-id3 0.2.6 - MP3 ID3 tag embedding for metadata (title, track number, cover art)
- filenamify 6.0.0 - Cross-platform filename sanitization
- multer 1.4.5-lts.1 - Multipart form data handling (optional, for potential future uploads)
- cors 2.8.5 - Cross-Origin Resource Sharing for API access
- morgan 1.10.0 - HTTP request logging
- dotenv 16.4.5 - Environment variable loading
- uuid 9.0.0 - Session ID generation for download tracking
- p-limit 4.0.0 - Parallel download concurrency control
- scroll-into-view-if-needed 3.0.6 - Smooth scroll to active download item
## Configuration
- `.env` file (server-side) - Not tracked in git
- Example at `web-version/.env.example` with defaults:
- `client/vite.config.ts` - Vite configuration with React plugin
- `client/tsconfig.json` - TypeScript compiler options (if present)
- `.replit` - Replit deployment config with Node.js 20 module
## Platform Requirements
- Node.js 20 or higher
- npm for dependency management
- Unix shell for build scripts (build.sh)
- Google Cloud Run (configured via `.replit` deploymentTarget)
- Node.js 20 runtime
- File system for temporary directory `/temp` (1-hour cleanup, 24-hour max age)
- Environment variable: `NODE_ENV=production`
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns
- Backend routes: lowercase with hyphens or `.js` extension. Example: `playlist.js`, `download.js`, `settings.js`
- Frontend components: PascalCase with `.tsx` extension. Example: `SimpleSettingsModal.tsx`, `ThemeToggle.tsx`, `StatusIcon.tsx`
- Service files: PascalCase with `.ts` extension. Example: `Suno.ts`, `Logger.ts`, `Utils.ts`, `SettingsManager.ts`
- Utility files: lowercase with `.js` extension. Example: `fileManager.js`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MrAshRhodes/suno-playlist-downloader-web](https://github.com/MrAshRhodes/suno-playlist-downloader-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
