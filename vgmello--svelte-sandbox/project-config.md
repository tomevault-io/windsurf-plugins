---
trigger: always_on
description: **NEVER IMPLEMENT CODE WITHOUT IMMEDIATELY TESTING IN BROWSER**
---

# Project Memory

# 🚨 CRITICAL DEVELOPMENT RULE 🚨
**NEVER IMPLEMENT CODE WITHOUT IMMEDIATELY TESTING IN BROWSER**
- Test EVERY single change in the browser before proceeding
- If you write ANY code, you MUST verify it works in the browser
- If you cannot test (server down, etc.), STOP and ask user to start server
- NO EXCEPTIONS - this is mandatory for ALL code changes

## Implementation Workflow

⚠️ **CRITICAL REQUIREMENT: ALWAYS TEST IN BROWSER DURING DEVELOPMENT** ⚠️

1. **Before starting work**: 
   - Ensure you understand the requirements
   - Confirm backend server is running on port 8101, if not request user to start it

2. **During development** (MUST DO AFTER EACH CHANGE):
   - Follow the established patterns and conventions
   - Use existing components and utilities
   - Write clean, maintainable code
   - **🚨 MANDATORY: Test EVERY change immediately in the browser**
   - **🚨 MANDATORY: Navigate to the relevant page and verify functionality works**
   - **🚨 MANDATORY: Test user interactions, form submissions, and UI behavior**
   - **🚨 MANDATORY: Check browser console for any errors**
   - **🚨 MANDATORY: Do NOT proceed to next change until current change is verified working**

3. **After implementing ALL changes**:
   - **FINAL VERIFICATION: Re-test all modified pages in the browser**
   - Verify mobile responsiveness if applicable
   - Confirm no console errors exist

4. **Before committing**:
   - Run `pnpm check` to ensure code quality, fix bugs
   - Run `pnpm format` to ensure code formatting, fix formatting issues
   - Run tests if applicable
   - Review changes for quality

5. **Git workflow**:
   - Commit changes with descriptive messages
   - Rebase if needed to keep history clean
   - Push changes to remote repository

## Core Architecture Principles

This application follows a **vertical architecture** with **minimal abstractions** for **maximum developer experience**:

### 🎯 Minimal Abstractions
- **Use OpenAPI-generated types directly** - no custom wrappers or API client classes
- **Direct openapi-fetch calls** in actions - no intermediate API layers  
- **Minimal custom interfaces** 

### 🏗️ Vertical Feature Organization  
- Each feature is **self-contained** in `src/lib/app/{feature}/`
- **High cohesion** within features, **loose coupling** between features
- Feature exports are **barreled** through `index.ts` for clean imports

### ⚡ Developer Experience Focus
- **Straightforward patterns** - easy to learn, easy to follow
- **Consistent error handling** with simple `{data, error}` structure  
- **Type-safe** throughout with TypeScript and Zod validation
- **Minimal cognitive overhead** - few concepts to learn
- **Vertical architecture** - business logic and UI clearly separated

### 🚀 Easy Feature Addition
Adding a new feature follows a **predictable, mechanical pattern**:
1. **Business Logic**: Create in `lib/app/{feature}/`
   - Define types in `{feature}-api.ts` (import from OpenAPI)
   - Create actions in `actions/` (use openapi-fetch directly)
   - Add configuration in `config.ts`
   - Export through `index.ts` (barrel exports)
2. **User Interface**: Create in `routes/{feature}/`
   - Build components in `components/` (route-specific)
   - Create pages with loaders/actions using feature business logic
   - Co-locate UI components with routes that use them

## Code style

- Use kebab-case for file naming
- Run `pnpm format` before you commit

## UI Components

- DO NOT change any files directly in the `lib/components/ui` folder - these are shadcn-svelte components
- Install new shadcn components using `pnpm dlx shadcn-svelte@latest add [component name] -y -o`.
- Component documentation is available via context7 mcp. Library: huntabyte/shadcn-svelte
- Use existing shadcn-svelte components when possible

## UI Conventions

- Always use `import type { PageProps } from './$types';` for +page.svelte files, when properties are needed;
- Always try to use the types from $types if available for +server.ts and +page.ts
- Always use a interface to declare svelte component props

```typescript [cashier-card.svelte]
interface Props {
	cashier: GetCashiersResult;
	ondeleted: (item: GetCashiersResult) => void;
}
```

## Error Handling Pattern

Actions use openapi-fetch directly and return a simplified Result type with `{data, error}` structure:

```typescript
import { ok, error, type ActionResult } from '$lib/utils/action-result';
import client from '$lib/api/billing';
import type { ZodError } from 'zod';

// Error result for all operations  
type ActionError = {
	error: string;
	code: number;
	validationErrors?: Record<string, string[]>;
};

// Query pattern - returns data
export const getItemQuery = async (id: string): Promise<ActionResult<Item>> => {
	if (!id?.trim()) {
		return error('Item ID is required', 400);
	}

	const { data, response } = await client.GET('/Items/{id}', {
		params: { path: { id } }
	});

	if (data) {
		return ok(data);
	}

	if (response?.status === 404) {
		return error('Item not found', 404);
	}
	return error('Failed to load item. Please try again later.', response?.status || 500);
};

// Command pattern - returns data 
export const createItemCommand = async (request: CreateItemRequest): Promise<ActionResult<Item>> => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vgmello) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
