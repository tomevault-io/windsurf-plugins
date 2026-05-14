---
trigger: always_on
description: **The goal is production-quality code, not a prototype**. Every change should be something you'd be confident shipping. Quality over speed. Completeness over convenience.
---

# Lanyards Development Guide

## THE GOLDEN RULE

**The goal is production-quality code, not a prototype**. Every change should be something you'd be confident shipping. Quality over speed. Completeness over convenience.

---

## Instructions for Claude

### Issue Tracking
Use GitHub Issues via the `gh` CLI for all task tracking:
- **NEVER** use markdown files for to-do lists or tracking work
- **ALWAYS** create issues for bugs and features before starting work
- Reference issue numbers in commits and PRs

### Git Workflow
- **NEVER** commit without explicit user instruction
- **NEVER** push without explicit user instruction
- **NEVER** use `--force` or destructive git commands
- You may run `git status`, `git diff`, `git log` freely
- You may stage files with `git add` when explicitly asked
- Leave version control decisions to the user

### Core Principles
**1. No Stubs, No Shortcuts**
- **NEVER** use placeholder implementations or `// TODO` comments
- **NEVER** skip functionality because it seems complex
- **NEVER** leave incomplete code paths
- Every function must be fully implemented and working

**2. Complete Before Moving On**
- Finish the current task before starting another
- If blocked, discuss with the user rather than working around it
- Each increment of work must be complete and functional

**3. Verify Your Work**
- Run `npm run lint` after changes
- Test locally with `npm run dev`
- Check the browser - don't assume it works

---

## Tech Stack

- **Framework**: Next.js (App Router) + TypeScript
- **Styling**: Tailwind CSS (no inline styles)
- **Auth**: AT Protocol OAuth (`@atcute/oauth-browser-client`)
- **Protocol**: AT Protocol (`@atproto/*` packages)
- **Data**: ProfileRepository pattern wrapping AtpAgent

---

## ⚠️ CRITICAL OAuth Configuration Rules

**NEVER USE `localhost` - ALWAYS USE `127.0.0.1`**

RFC 8252 REQUIRES loopback IP addresses, NOT hostnames:
- ✅ `http://127.0.0.1:3000/oauth/callback`
- ❌ `http://localhost:3000/oauth/callback`

AT Protocol OAuth will reject `localhost` with "invalid_request" error.

**OAuth Client Configuration:**
- Local development (`http://127.0.0.1`): Uses RFC 8252 loopback client format
- Production/Staging (`https://`): Uses metadata URL format
- Base URL is automatically derived from request - no environment variable needed
- `SERVER_HOST = '127.0.0.1'` in `next.config.ts`
- Access the app via `http://127.0.0.1:3000` (NOT localhost)

---

## Project Structure

```
src/
├── app/              # Next.js routes
│   ├── [handle]/     # Public profile pages
│   ├── api/          # REST endpoints
│   ├── auth/         # Login pages
│   └── dashboard/    # Protected routes
├── components/       # React components (strict 4-file structure)
├── lib/
│   ├── auth/         # Legacy session compatibility layer
│   ├── oauth/        # OAuth client and session management
│   ├── data/         # ProfileRepository, DOI resolution
│   └── utils.ts      # Utilities including cn()
└── types/            # TypeScript definitions
    └── generated/    # Auto-generated from lexicons

lexicons/             # AT Protocol schemas (*.json)
docs/                 # Hugo documentation site
```

---

## Code Quality Standards

### TypeScript
- Strict mode is enabled - respect it
- Use proper types, avoid `any` (warnings are configured)
- Prefer type inference where obvious, explicit types for function signatures
- Use `Result`-style error handling patterns where appropriate

### Component Architecture
**All components MUST follow this 4-file structure:**

```
ComponentName/
├── ComponentName.tsx           # Logic and JSX
├── ComponentName.types.ts      # TypeScript interfaces
├── ComponentName.styles.ts     # Tailwind class strings
└── ComponentName.constants.ts  # Hardcoded values (optional)
```

### Styling Rules
- **No margin utilities** - use `gap` for spacing between siblings, `padding` for internal
- **All text needs `leading-*`** - always specify line-height
- **Use `cn()`** from `@/lib/utils` for conditional classes
- **No inline styles** - Tailwind only

### Code Conventions
- **Quotes**: Single quotes
- **Semicolons**: Yes
- **Trailing commas**: ES5 style
- **Line width**: 80 characters
- **Unused vars**: Prefix with `_`

### Error Handling
- Use try/catch in API routes with meaningful error responses
- Return appropriate HTTP status codes
- Never swallow errors silently
- Log errors server-side for debugging

---

## Form Standards

All forms must be accessible, usable, and provide clear feedback.

### Input Selection by Data Type

Choose the appropriate input for the data:

| Data Type               | Input Component                      | Example                      |
| ----------------------- | ------------------------------------ | ---------------------------- |
| Free text (short)       | `<input type="text">`                | Name, title                  |
| Free text (long)        | `<textarea>`                         | Bio, description             |
| Closed list (large)     | Select with typeahead/autocomplete   | Country, institution         |
| Closed list (small, ≤5) | Radio buttons                        | Honorific (Dr, Prof, Mr, Ms) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renderghost/lanyards](https://github.com/renderghost/lanyards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
