---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Standard Workflow
1. First think through the problem, read the codebase for relevant files, and write a plan to todo.md.
2. The plan should have a list of todo items that you can check off as you complete them
3. Before you begin working, check in with me and I will verify the plan.
4. Then, begin working on the todo items, marking them as complete as you go.
5. Please every step of the way just give me a high level explanation of what changes you made
6. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
7. Finally, add a review section to the todo.md file with a summary of the changes you made and any other relevant information.

## Development Commands

```bash
# Start development server
npm start

# Build for production
npm run build

# Run tests
npm test

# Run linting
npm run lint
npm run lint:fix
```

## Architecture Overview

### Core Technology Stack
- **React 18** with functional components and hooks
- **Firebase** for authentication, Firestore database, and file storage
- **React Router** for client-side routing
- **Lucide React** for icons
- **Bootstrap** for base styling with custom CSS variables

### Application Structure

**Authentication Flow:**
- `AuthProvider` manages global auth state via `AuthContext`
- Firebase Authentication with organization-based multi-tenancy
- Role-based access control (admin, photographer, etc.)
- Protected routes require authentication

**Layout Architecture:**
- `Layout` component contains fixed `Sidebar` and `Header`
- Main content area dynamically loads page components
- Sidebar navigation items defined in `/src/components/layout/Sidebar.js`

**Context Providers:**
- `AuthContext` - user authentication, profile, and organization data
- `JobsContext` - sports job management and data operations  
- `ChatContext` - real-time messaging with cache-first loading (REFERENCE IMPLEMENTATION)
- `ToastContext` - global notification system
- `DataCacheContext` - centralized cache management and read tracking

**Modal Rendering:**
- Modals use `ReactDOM.createPortal` to render at document.body level
- Critical: Use inline styles for overlay positioning to avoid CSS conflicts
- Modal overlay must have `position: fixed`, `zIndex: 10001`, and flexbox centering
- Inner modal container needs `position: relative`, `margin: 0`, `transform: none`

### Key Modules

**Sports Module (`/src/components/sports/`):**
- Comprehensive sports photography job management
- File upload and roster management with Excel/CSV support
- Real-time job tracking and statistics
- Player search across jobs and rosters

**Firebase Integration:**
- Configuration in `/src/firebase/config.js` with environment variable fallbacks and offline persistence
- Firestore collections: users, organizations, schools, sportsJobs, conversations, messages
- File storage for user photos and roster uploads  
- Real-time listeners with cache optimization (see `ChatContext` for reference implementation)
- ReadCounter system for monitoring Firebase usage and costs (`/src/services/readCounter.js`)
- Cache services for efficient data management (`/src/services/chatCacheService.js` as reference)

**Styling System:**
- CSS custom properties defined in `/src/styles/variables.css`
- Component-specific CSS files follow naming convention: `ComponentName.css`
- Global styles in `/src/styles/globals.css`
- Sports module has dedicated styling in `/src/styles/sports/`

### Data Flow Patterns

**Cache-First Data Loading (STANDARD PATTERN):**
- Load from cache immediately for instant display
- Fetch updates from Firestore in background  
- Use optimized real-time listeners for incremental updates
- Track all operations with ReadCounter for cost monitoring
- Reference implementation: `ChatContext` and `ChatCacheService`

**User Profile Management:**
- User profile linked to organization via `organizationID`
- Profile photos with crop functionality and multiple image formats
- Settings modals for both user profile and studio/organization settings
- **REQUIRED**: Implement caching for user profile data

**Job Management (Sports):**
- Jobs associated with schools and organizations
- Roster data uploaded via Excel/CSV with real-time preview
- Job status tracking (active, completed, archived)
- Search functionality across jobs and player rosters
- **REQUIRED**: Implement cache-first loading for job lists and search results

**Real-Time Messaging (REFERENCE IMPLEMENTATION):**
- Cache-first message loading with instant display
- Optimized listeners that only fetch new messages
- Local storage persistence with automatic cleanup
- ReadCounter integration for cost tracking
- Cross-platform data synchronization ready

**File Upload Patterns:**
- User profile photos: original + cropped versions stored
- Roster files: Excel/CSV processing with validation and preview
- All uploads use Firebase Storage with organized folder structure

### CSS Architecture Notes

**Modal Positioning:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jsunwilke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
