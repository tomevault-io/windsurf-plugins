---
trigger: always_on
description: This document contains critical information for working with the pbjs_engine Prebid wrapper platform. Follow these guidelines precisely.
---

# Development Guidelines for pbjs_engine

This document contains critical information for working with the pbjs_engine Prebid wrapper platform. Follow these guidelines precisely.

## Project Overview

pbjs_engine is a Prebid.js wrapper platform that provides server-managed configurations for publishers. The platform enables publishers to embed a lightweight JavaScript wrapper on their sites that automatically loads Prebid.js with centrally managed bidder configurations.

### Technology Stack

**Backend (API Server):**
- Node.js 20 LTS with TypeScript
- Fastify web framework
- Drizzle ORM for type-safe database access
- SQLite for data storage (PostgreSQL-compatible schema)
- JWT authentication with bcrypt password hashing

**Frontend (Admin Portal):**
- React 18 with TypeScript
- Vite for build tooling
- Tailwind CSS for styling
- Zustand for state management
- React Router for navigation

**Publisher Wrapper:**
- TypeScript compiled to ES2015
- Webpack with Terser for minification
- Lightweight (<2KB gzipped) JavaScript library
- CDN-ready with cache control headers

### Architecture

```
pbjs_engine/
├── apps/
│   ├── api/              # Fastify backend server
│   │   ├── src/
│   │   │   ├── db/       # Database schema and migrations
│   │   │   ├── routes/   # API route handlers
│   │   │   └── index.ts  # Server entry point
│   │   └── data/         # SQLite database files
│   ├── admin/            # React admin portal
│   │   ├── src/
│   │   │   ├── components/  # UI components
│   │   │   ├── pages/       # Page components
│   │   │   ├── stores/      # Zustand state stores
│   │   │   └── App.tsx
│   │   └── dist/         # Production build output
│   └── wrapper/          # Publisher wrapper script
│       ├── src/
│       │   └── pb.ts     # Wrapper source code
│       └── dist/         # Minified production build
└── docs/                 # Documentation
```

## Core Development Rules

### 1. Language & Framework Standards

**Backend:**
- **Language**: TypeScript 5.3+ with strict mode enabled
- **Runtime**: Node.js 20 LTS
- **Web Framework**: Fastify for high-performance HTTP handling
- **Code Style**: Prettier for formatting, ESLint for linting
- **Error Handling**: Use try-catch with proper error responses
- **Testing**: Vitest for unit and integration tests
- **Naming**: Follow conventions in `NAMING_CONVENTIONS.md`

**Frontend:**
- **Language**: TypeScript with React JSX
- **Framework**: React 18 with functional components and hooks
- **Styling**: Tailwind CSS utility classes
- **State**: Zustand for global state, React hooks for local state
- **Forms**: Controlled components with validation
- **Naming**: Follow conventions in `NAMING_CONVENTIONS.md`

### 2. Database Guidelines

**Technology**: SQLite with Drizzle ORM

**Schema Design:**
- Use proper foreign key constraints
- Implement soft deletes with `deleted_at` timestamp
- Use UUIDs for primary keys
- Maintain audit fields: `created_at`, `updated_at`, `deleted_at`

**Data Hierarchy:**
```
Publisher → Website → Ad Unit
          ↓
       Bidder Configuration
```

**Migrations:**
```bash
# Create migration script in apps/api/src/db/
# Run with: npm run migrate
```

**Query Patterns:**
```typescript
// Use Drizzle ORM for type safety
import { db } from '../db';
import { publishers, websites, adUnits } from '../db/schema';
import { eq, and, isNull } from 'drizzle-orm';

// Fetch with soft delete filter
const activePublishers = db.select()
  .from(publishers)
  .where(isNull(publishers.deletedAt))
  .all();

// Join query
const websiteWithAdUnits = db.select()
  .from(websites)
  .leftJoin(adUnits, eq(websites.id, adUnits.websiteId))
  .where(eq(websites.publisherId, publisherId))
  .all();
```

### 3. API Design

**RESTful Conventions:**
- Use plural nouns for resources (`/publishers`, `/websites`)
- Hierarchical routes for relationships (`/websites/:id/ad-units`)
- Proper HTTP methods (GET, POST, PUT, DELETE)
- Use HTTP status codes correctly (200, 201, 400, 401, 404, 500)

**Authentication:**
```typescript
// JWT-based authentication
// Token required in Authorization header: Bearer <token>
// Role-based access: super_admin, admin, publisher
```

**Route Structure:**
```
/api/auth/login              # Authentication
/api/auth/forgot-password    # Password reset
/api/publishers              # Publisher CRUD
/api/publishers/:id          # Single publisher
/api/websites/:id/ad-units   # Hierarchical ad units
/api/system/health           # System monitoring
/pb.min.js                   # Wrapper script (no /api prefix)
/c/:publisherId              # Public config endpoint
```

**Response Format:**
```typescript
// Success
{ data: T, message?: string }

// Error
{ error: string, details?: any }

// Pagination
{ data: T[], total: number, page: number, limit: number }
```

### 4. Frontend Patterns

**Component Structure:**
```typescript
// Functional component with TypeScript
interface Props {
  id: string;
  onSave: (data: FormData) => void;
}

export function ComponentName({ id, onSave }: Props) {
  const [state, setState] = useState<StateType>(initialState);

  useEffect(() => {
    // Side effects
  }, [dependencies]);

  return (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [q9void/prebidjs-light](https://github.com/q9void/prebidjs-light) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
