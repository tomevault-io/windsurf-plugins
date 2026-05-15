---
trigger: always_on
description: This is a teaching demo showcasing the Sogni AI render network platform through a clean, production-ready full-stack application. The app generates tattoo concept art using AI rendering with real-time progress streaming.
---

# Sogni Demo Superapp - Cursor AI Rules
# A Node.js Express + React + TypeScript + Sogni SDK demo application

## Project Overview
This is a teaching demo showcasing the Sogni AI render network platform through a clean, production-ready full-stack application. The app generates tattoo concept art using AI rendering with real-time progress streaming.

## Stack & Architecture
- **Backend**: Node.js + Express (ES modules), Sogni Client SDK, Server-Sent Events
- **Frontend**: React 18 + TypeScript + Vite, responsive design with accessibility
- **Package Manager**: pnpm workspace (monorepo structure)
- **Communication**: REST API + SSE for real-time streaming
- **Styling**: Inline CSS with design system variables, mobile-first responsive

## Core Principles
1. **Keep it simple and teachable** - This is a demo/teaching app, prioritize clarity over complexity
2. **Maintain clean separation** - Backend handles Sogni SDK, frontend handles UX
3. **Fire-and-stream pattern** - POST to start, SSE to stream progress/results
4. **Production-ready patterns** - Even though it's a demo, use production best practices

## File Structure Rules
```
sogni-demo-superapp/
├── server/           # Express API (Sogni SDK integration)
│   ├── index.js     # Main server file (ES modules)
│   ├── package.json # Server dependencies
│   └── .env         # Server environment (gitignored)
├── web/             # React frontend
│   ├── src/
│   │   ├── main.tsx    # App entry point
│   │   ├── App.tsx     # Main component (keep monolithic for teaching)
│   │   └── components/ # Create this when App.tsx > 500 lines
│   ├── package.json    # Frontend dependencies
│   └── .env           # Frontend environment (gitignored)
└── package.json      # Root workspace config
```

## Coding Standards

### General
- Use 2-space soft tabs consistently across all files
- Prefer explicit imports over default exports for better IDE support
- Add JSDoc comments for complex functions and API endpoints
- Keep functions focused and under 50 lines when possible

### TypeScript/JavaScript
- Use TypeScript strict mode in frontend
- Use ES modules syntax consistently (`import`/`export`)
- Prefer `const` over `let`, avoid `var`
- Use meaningful variable names that explain intent
- Type React components properly with interfaces for props

### React Patterns
- Keep App.tsx as main component until it exceeds 500 lines
- Use functional components with hooks
- Implement proper error boundaries for production readiness
- Use React.memo() for expensive components
- Maintain accessibility with proper ARIA attributes and semantic HTML

### Backend Patterns
- Keep server/index.js as single file until it exceeds 800 lines
- Use proper error handling with try/catch blocks
- Implement CORS properly for security
- Use environment variables for all configuration
- Follow REST conventions for API endpoints

### State Management
- Use React useState for simple state
- Consider useReducer for complex state transitions
- Keep state close to where it's used
- Use SSE for real-time updates, not polling

## Context Window Management

### File Size Guidelines
- **Critical**: Split files when they exceed these limits:
  - React components: 500 lines → create components/ subdirectory
  - Server files: 800 lines → create routes/, middleware/, utils/ subdirectories
  - Any file: 1000 lines → mandatory split

### Code Organization
- Group related functionality together
- Extract reusable utilities to separate files
- Use barrel exports (index.js) for clean imports
- Keep configuration at the top of files

### When Splitting Components
```typescript
// Before (App.tsx > 500 lines)
export default function App() { /* ... */ }

// After
// src/components/TattooForm.tsx
// src/components/IdeaGrid.tsx
// src/components/ProgressStream.tsx
// src/App.tsx (orchestrates components)
```

### When Splitting Server
```javascript
// Before (server/index.js > 800 lines)

// After
// server/routes/generate.js
// server/routes/progress.js
// server/middleware/cors.js
// server/utils/sogni.js
// server/index.js (orchestrates modules)
```

## API Design Patterns

### Endpoints
- `POST /api/generate` - Start render with Sogni SDK
- `GET /api/progress/:projectId` - SSE stream for real-time updates
- `GET /api/cancel/:projectId` - Cancel running project
- `GET /api/result/:projectId/:jobId` - Proxy result images
- `GET /api/health` - Health check

### Error Handling
- Always return consistent error format: `{ error: string }`
- Use appropriate HTTP status codes
- Handle Sogni SDK errors gracefully
- Log errors server-side, never expose internal details to client

### SSE Best Practices
- Send heartbeat every 20 seconds to prevent proxy timeouts
- Clean up connections properly on client disconnect
- Handle connection drops gracefully with reconnection logic
- Use structured event format: `{ type: string, ...data }`

## Sogni SDK Integration

### Authentication
- Store credentials in server environment only
- Never expose API keys to frontend
- Use lazy client initialization to avoid startup issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blkluv/sogni-demo-superapp](https://github.com/blkluv/sogni-demo-superapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
