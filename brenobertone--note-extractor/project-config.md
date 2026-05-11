---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SnapFlow is an AI-powered note extraction application that transforms images into categorized digital records. It uses OpenAI's GPT-4o vision model to extract text from images and automatically categorizes content as "Tasks" or "Habits", persisting to a Supabase PostgreSQL backend.

## Development Commands

### Setup

```bash
npx supabase start              # Start local Supabase (Docker required)
npm install                     # Install dependencies
```

### Development

```bash
npm run dev                     # Start Next.js dev server (http://localhost:3000)
npm run build                   # Production build
```

### Testing

```bash
npm run test                    # Unit tests (mocked, fast)
npm run test:watch              # Unit tests in watch mode
npm run test:integration        # Integration tests (requires Supabase Docker)
npx playwright test             # E2E tests (auto-starts dev server)
```

### Code Quality

```bash
npm run lint                    # ESLint
npm run type-check              # TypeScript validation
npm run format:write            # Auto-fix formatting issues
npm run format:check            # Check formatting
```

## Architecture

### Core Flow

1. User uploads image(s) via `UploadComponent` (frontend)
2. POST to `/api/process` with FormData
3. API route converts images to base64, sends to OpenAI GPT-4o
4. LLM returns JSON: `{content: string, category: "Tasks"|"Habits"}`
5. Data persisted to `user_actions` table in Supabase
6. Images uploaded to Supabase Storage (`note-images` bucket)
7. Image metadata stored in `images` table with foreign key to `user_actions`
8. Response returned to frontend, gallery auto-refreshes
9. GET from `/api/actions` retrieves actions with nested images and public URLs
10. `ThumbnailGrid` displays clickable thumbnails in 4-column grid
11. `ImageModal` shows full-size images with extracted text
12. User can hover over thumbnails to reveal delete button
13. DELETE to `/api/actions/[id]` removes images from storage and cascades database deletion

### Key Files

- **API Routes**:
  - `src/app/api/process/route.ts` - POST endpoint for image processing and storage
  - `src/app/api/actions/route.ts` - GET endpoint for retrieving actions with images
  - `src/app/api/actions/[id]/route.ts` - DELETE endpoint for removing actions and images
- **Database Schema**:
  - `supabase/migrations/20260323000000_create_user_actions.sql` - User actions table
  - `supabase/migrations/20260325000000_add_images_table.sql` - Images table + storage bucket
- **Frontend Components**:
  - `src/components/UploadComponent.tsx` - File upload with multi-image support
  - `src/components/ThumbnailGrid.tsx` - 4-column gallery with auto-refresh and delete handler
  - `src/components/ThumbnailCard.tsx` - Individual thumbnail with category label and delete button (visible on hover)
  - `src/components/ImageModal.tsx` - Full-screen modal with image carousel
  - `src/app/page.tsx` - Main page integrating all components
- **Types**: `src/types/index.ts` - TypeScript interfaces for `Image` and `UserAction`
- **Unit Tests**:
  - `src/__tests__/api/process.test.ts` - Process API with storage mocks
  - `src/__tests__/api/actions.test.ts` - Actions API with pagination tests
  - `src/__tests__/api/delete-action.test.ts` - Delete API with cascade deletion
  - `src/__tests__/components/*.test.tsx` - Component tests (includes delete button tests)
- **Integration Tests**:
  - `src/__tests__/integration/database.test.ts` - Database operations
  - `src/__tests__/integration/storage.test.ts` - Storage bucket operations
  - `src/__tests__/integration/delete-action.test.ts` - Delete with cascade verification
- **E2E Tests**:
  - `e2e/upload.spec.ts` - Basic upload flow
  - `e2e/multi-upload.spec.ts` - Multi-image upload
  - `e2e/thumbnail-gallery.spec.ts` - Gallery interactions and modal (10 tests)

### Database

- Local Supabase stack runs via Docker (`npx supabase start`)
- Schema defined in SQL migrations under `supabase/migrations/`
- **Tables**:
  - `user_actions`: id, content, category, created_at
  - `images`: id, user_action_id (FK), storage_path, file_name, mime_type, file_size, created_at
- **Storage**: `note-images` bucket (public, 10MB limit) created automatically by migration
- RLS enabled with permissive policies for development
- Reset database: `npx supabase db reset` (recreates tables + bucket)

### Testing Strategy

Three test layers with different scopes:

1. **Unit Tests** (`npm run test`): Mock external services (Supabase, OpenAI). Fast feedback.
2. **Integration Tests** (`npm run test:integration`): Real Supabase connection. Validates DB operations.
3. **E2E Tests** (`npx playwright test`): Full browser flow. Validates UI interactions.

**Test Exclusion**: Vitest config excludes `e2e/` directory; Playwright config targets `e2e/` only.

## Development Workflow

### Test-Driven Development (TDD)

**Always write tests before implementation**. Pattern:

1. Write test in `src/__tests__/` (unit or integration)
2. Run test to verify it fails
3. Implement minimal code to pass
4. Refactor if needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brenobertone/Note-Extractor](https://github.com/brenobertone/Note-Extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
