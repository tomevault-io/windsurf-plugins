---
trigger: always_on
description: Provides sidebar navigation and consistent structure for authenticated pages. Use it with:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Canvasly**, a portfolio CMS built with Nuxt 4, Vue 3, and Supabase. It allows users to manage portfolio projects, blocks (project components), and social links through a dashboard interface.

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Generate static site
npm generate
```

## Architecture Overview

### Stack
- **Frontend**: Nuxt 4.1.3 + Vue 3.5.22
- **Backend**: Nuxt Server Routes (H3 framework)
- **Database**: Supabase (PostgreSQL)
- **Auth**: Supabase Auth (JWT-based, httpOnly cookies)
- **State**: Pinia stores
- **UI**: shadcn-nuxt + Tailwind CSS
- **Storage**: Supabase Storage (for images)

### Directory Structure

```
/app                    # Frontend application
  ├── pages/           # File-based routes (auto-routed by Nuxt)
  ├── components/      # Vue components (auto-imported)
  ├── stores/          # Pinia state management
  ├── types/           # TypeScript interfaces
  ├── layouts/         # Page layouts
  └── lib/             # Utilities

/server                 # Backend API
  ├── api/            # REST API endpoints (file-based routing)
  └── utils/          # Server utilities (e.g., Supabase client)
```

## Data Models & Relationships

### Database Schema

```
User (Supabase Auth)
├── Projects (1:N)
│   ├── Blocks (1:N)
│   └── Categories (M:N via projectCategory)
└── Links (1:N)
```

**Key Tables**:
- `auth.users`: Managed by Supabase Auth
  - Custom field: `user_metadata.display_name`
- `project`: User's portfolio projects
  - Fields: `id`, `user_id`, `title`, `description`, `thumbnail`, timestamps
- `block`: Components/sections of a project
  - Fields: `id`, `project_id`, `title`, `description`, `url`
- `link`: Social/external links
  - Fields: `id`, `user_id`, `title`, `url`, `icon`
- `categories`: Project categories
- `projectCategory`: Junction table for M:N relationship

**Storage Buckets**:
- `projects`: Project thumbnail images
- `icons`: Link icon images

## Authentication System

### How It Works

1. **Registration/Login**: User submits credentials → API handler calls Supabase Auth → Returns JWT tokens
2. **Token Storage**: Access and refresh tokens stored in **httpOnly cookies** (`sb-access-token`, `sb-refresh-token`)
3. **API Protection**: All API routes (except auth endpoints) validate tokens via `_authGard.ts` middleware
4. **Session Management**: Tokens automatically refreshed by Supabase; cookies updated via auth state listener

### Auth Guard Pattern

**File**: [server/api/_authGard.ts](server/api/_authGard.ts)

The `_` prefix makes this a global middleware. It:
- Whitelists `/api/auth/login` and `/api/auth/register`
- Extracts tokens from cookies
- Validates with `supabase.auth.getUser()`
- Stores authenticated user in `event.context.auth`
- Returns 401 if unauthorized

**Usage in API routes**:
```typescript
import { authGuard } from '~/server/api/_authGard'

export default defineEventHandler(async (event) => {
  const user = await authGuard(event)  // Throws 401 if not authenticated
  // ... use user.id for queries
})
```

## API Structure & Conventions

### File-Based Routing

API routes use HTTP method as file extension:
- `index.get.ts` → GET
- `index.post.ts` → POST
- `index.patch.ts` → PATCH
- `index.delete.ts` → DELETE
- `[id]/index.get.ts` → GET with dynamic parameter

### Standard Patterns

**Reading Request Data**:
```typescript
import { readBody, readMultipartFormData, getRouterParam } from 'h3'

// JSON body
const body = await readBody(event)

// File uploads
const formData = await readMultipartFormData(event)

// Route params
const id = getRouterParam(event, 'id')
```

**Response Format**:
```typescript
// Success
return { data: result }

// Error
throw createError({
  statusCode: 400,
  message: 'User-friendly message'
})
```

**File Upload Pattern**:
```typescript
// 1. Read multipart form data
const formData = await readMultipartFormData(event)
const file = formData.find(f => f.name === 'thumbnail')

// 2. Upload to Supabase Storage
const { data, error } = await supabase.storage
  .from('bucket-name')
  .upload(`path/${file.filename}`, file.data)

// 3. Get public URL
const { data: { publicUrl } } = supabase.storage
  .from('bucket-name')
  .getPublicUrl(data.path)

// 4. Store URL in database
```

**Ownership Verification**:
Always filter by `user_id` to ensure users can only access their own resources:
```typescript
const { data } = await supabase
  .from('table')
  .select('*')
  .eq('user_id', user.id)  // Critical: prevents access to other users' data
```

## State Management (Pinia)

### Store Pattern

All stores follow this structure:
```typescript
import { defineStore } from 'pinia'

export const useStoreName = defineStore('storeName', {
  state: () => ({
    items: [],
    loading: false,
    error: null
  }),

  getters: {
    // Computed properties
  },

  actions: {
    // Use $fetch() for API calls (auto-includes cookies)
    async fetchItems() {
      this.loading = true
      try {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kicksbld) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
