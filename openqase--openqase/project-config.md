---
trigger: always_on
description: This file contains development guidelines and practices specific to the OpenQase project. Claude Code should follow these guidelines when working on the codebase.
---

# Claude Code Development Guidelines for OpenQase

This file contains development guidelines and practices specific to the OpenQase project. Claude Code should follow these guidelines when working on the codebase.

## CHANGELOG Maintenance

### When to Update CHANGELOG.md
Update the CHANGELOG for these types of changes:
- **Fixed**: Bug fixes that affect user experience or content display
- **Added**: New features, components, or significant functionality
- **Changed**: Modifications to existing features that change behavior
- **Security**: Fixes for security vulnerabilities or content exposure issues
- **Removed**: Features or functionality that has been removed

### CHANGELOG Update Process
1. **During development**: Add entries to the `[Unreleased]` section
2. **Before major commits**: Ensure CHANGELOG reflects the changes being committed
3. **Format**: Use clear, user-focused descriptions that explain the impact, not just the technical details

### CHANGELOG Entry Examples
```markdown
### Fixed
- **CMS Content Filtering**: Fixed unpublished case studies appearing on public pages

### Added
- **New Component**: Added particle field animation for homepage background

### Changed
- **Search Functionality**: Improved search performance and added type filtering
```

## Commit Practices

### Commit Message Format
- Use conventional commit format: `type: description`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Keep first line under 50 characters
- Add detailed explanation in body if needed

### Before Committing
1. Update CHANGELOG.md if the change is notable
2. Ensure code follows existing patterns and conventions
3. Test changes don't break existing functionality

## Content Management System (CMS) Guidelines

### Published Content Filtering
- Always filter relationships for `published: true` in runtime queries
- Preserve preview mode functionality for team access to drafts
- Maintain static site generation architecture for performance
- Never expose unpublished content in public queries

### Database Query Patterns
- Use `getStaticContentWithRelationships()` for single content items
- Use `getStaticContentList()` for content lists
- Include `published` field in relationship queries when filtering is needed
- Apply published filters conditionally based on preview mode

### Supabase Client Types
Two client factories exist in `src/lib/supabase-server.ts`. Use the right one for the context:

- **`createServiceRoleSupabaseClient()`** — Bypasses RLS. Required for:
  - Build-time static generation (no user session exists)
  - ISR revalidation (no user session exists)
  - Admin server actions (write operations)
  - This is safe because the service role key is server-only (`SUPABASE_SERVICE_ROLE_KEY`, never `NEXT_PUBLIC_`)

- **`createServerSupabaseClient()`** — Respects RLS, uses the current user's session. Use for:
  - Auth checks (verifying the user is logged in / is admin)
  - Any future user-scoped queries where RLS should apply

**Do not** use the service role client in client components or expose the key via `NEXT_PUBLIC_` env vars. See GitHub issue #144 for the full audit.

### Deletion System
- **Soft delete**: Use `soft_delete_content()` database function
- **Recovery**: Use `recover_content()` database function
- Content has `content_status` field: 'draft', 'published', 'archived', 'deleted'
- 30-day retention period for soft-deleted content before permanent deletion
- **Note**: The `public_*`, `admin_*`, and `trash_*` database views were planned but **do not exist**. Filtering is done via `.eq('published', true)` in queries and JS-level filtering.

## Architecture Principles

### Static Site Generation & Revalidation (ISR)

The site uses **static site generation with on-demand revalidation** — a two-layer caching strategy:

#### Layer 1: On-Demand Revalidation (primary)
When content is saved/published/unpublished in the admin CMS, server actions call `revalidatePath()` to immediately invalidate the affected pages:
- **Category listing pages** (e.g., `/case-study`, `/paths/algorithm`) are revalidated so new/removed items appear
- **Individual detail pages** (e.g., `/case-study/my-slug`) are revalidated so content changes appear
- Server actions are in `src/app/admin/*/[id]/actions.ts`

Every save/publish/unpublish action must revalidate:
1. The admin listing page (e.g., `/admin/case-studies`)
2. The public listing page (e.g., `/paths/algorithm` or `/case-study`)
3. The individual page by slug (e.g., `/paths/algorithm/${slug}`)

#### Layer 2: ISR Safety Net (fallback)
All dynamic `[slug]/page.tsx` files export `revalidate = 86400` (24 hours). This catches **cross-entity staleness** — when a related entity changes (e.g., an algorithm's name is updated), pages that display that algorithm won't be directly revalidated by the algorithm's save action. The 24-hour ISR ensures these pages refresh eventually while keeping Vercel ISR write usage low.

#### When Adding New Content Types
1. Create admin server actions with proper `revalidatePath()` calls for save/publish/unpublish
2. Add `export const revalidate = 86400` to the public `[slug]/page.tsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openqase/openqase](https://github.com/openqase/openqase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
