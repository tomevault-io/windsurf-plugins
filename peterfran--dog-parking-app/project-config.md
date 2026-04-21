---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the frontend application for the Dog Parking platform - a Next.js 15 React application providing premium dog care services including daycare, boarding, grooming, and training. The app integrates with a separate AWS serverless backend API and uses Firebase for authentication.

## Visual Development

### Design Principles

- Comprehensive design checklist in `/context/design-principles.md`
- Brand style guide in `/context/style-guide.md`
- When making visual (front-end, UI/UX) changes, always refer to these files for guidance

### Browser Testing & Debugging

**Available Tools:**
- **Chrome DevTools MCP** - For performance analysis, debugging, and visual inspection
- **Playwright MCP** - For automated testing and reliable screenshots

**When to Use:**
Use browser tools when the user requests:
- Visual review or screenshots
- Performance analysis (Core Web Vitals, LCP, CLS, FCP)
- Debugging (console errors, network issues)
- Responsive design verification
- Accessibility checks

**Do not** automatically run browser tools after every change. Wait for explicit user request.

### Comprehensive Design Review

Invoke the `@agent-design-review` subagent for thorough design validation when the user requests:

- Comprehensive review of UI/UX features
- PR review with visual changes
- Accessibility and responsiveness testing

## Development Commands

**Development Server:**

```bash
npm run dev          # Start development server with Turbopack
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint (disabled during builds)
```

**Environment Setup:**

```bash
cp .env.example .env.local   # Copy environment template
# Edit .env.local with Firebase configuration and API URL
```

**Key Environment Variables:**

- `NEXT_PUBLIC_FIREBASE_API_KEY` - Firebase API key
- `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` - Firebase auth domain
- `NEXT_PUBLIC_FIREBASE_PROJECT_ID` - Firebase project ID
- `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` - Firebase storage bucket
- `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` - Firebase messaging sender ID
- `NEXT_PUBLIC_FIREBASE_APP_ID` - Firebase app ID
- `NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID` - Firebase measurement ID (optional)
- `NEXT_PUBLIC_API_BASE_URL` - Backend API endpoint (AWS Lambda)
- `NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_HOST` - Development emulator (optional, use localhost:9099)

## Architecture Overview

### Authentication Architecture

- **Firebase Authentication** handles user registration, login, and JWT token management
- **AuthContext** (`src/contexts/AuthContext.tsx`) provides authentication state throughout the app
- **ProtectedRoute** component wraps authenticated pages and handles redirects
- JWT tokens are automatically included in API requests via the `apiClient`

### Data Flow Architecture

- **TanStack React Query** manages server state and API caching
- **useApi hooks** (`src/hooks/useApi.ts`) provide typed API methods with authentication
- **ApiClient** (`src/lib/api-client.ts`) handles HTTP requests with automatic token injection
- **Query keys** are centralized in `QueryKeys` constant for type-safe cache invalidation
- **Protected queries** automatically include Firebase JWT tokens and are disabled when user is not authenticated
- **Mutations** automatically invalidate related queries on success

### Styling Architecture

- **Tailwind CSS v4** is used for styling with CSS-based configuration
- **Custom color palette** defined in `src/app/globals.css` using the `@theme` directive
- **UI components** (`src/components/ui/`) use `class-variance-authority` for variant management with Tailwind classes
- Each UI component inlines the `cn` utility function to avoid import resolution issues
- Components use a warm, professional design with gentle gradients and thoughtful color choices
- **Design system colors**:
  - Primary: `#4f46e5` (warm indigo, friendlier than harsh navy)
  - Accent colors: `accent-sage` (`#dcfce7`), `accent-peach` (`#fed7aa`), `accent-lavender` (`#f3e8ff`)
  - Orange accent: `#f97316` (warmer than aggressive orange)
- **Important**: Tailwind v4 uses CSS-based config in `globals.css`, NOT `tailwind.config.ts`

### Tailwind CSS v4 Configuration

This project uses Tailwind CSS v4 (beta) which has a completely different configuration system:

**Color Configuration (in `src/app/globals.css`):**
```css
@theme {
  --color-primary: #4f46e5;
  --color-accent-sage: #dcfce7;
  --color-accent-peach: #fed7aa;
  --color-accent-lightBlue: #dbeafe;
  --color-accent-lavender: #f3e8ff;
  --color-accent-textGray: #6b7280;
  /* etc... */
}
```

**Key Differences from v3:**
- Configuration is CSS-based using `@theme` directive, not JavaScript config file
- Custom colors are defined as CSS variables in `globals.css`
- `tailwind.config.ts` is essentially empty (kept for compatibility only)
- PostCSS uses `@tailwindcss/postcss` plugin
- Build process compiles CSS variables into utility classes

**Adding New Colors:**
1. Add to `@theme` block in `globals.css` using `--color-` prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peterFran) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
