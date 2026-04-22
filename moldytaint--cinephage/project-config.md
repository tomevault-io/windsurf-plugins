---
trigger: always_on
description: Guidelines for agentic coding assistants working in the Cinephage codebase.
---

# AGENTS.md

Guidelines for agentic coding assistants working in the Cinephage codebase.

## Build/Lint/Test Commands

```bash
npm run dev              # Start dev server
npm run build            # Production build
npm start                # Run production server
npm run check            # TypeScript + Svelte type checking
npm run lint             # ESLint + Prettier validation
npm run lint:fix         # Auto-fix lint issues
npm run format           # Auto-format code with Prettier
npm run test             # Run all tests once
npm run test:unit        # Run tests in watch mode
npm run test:coverage    # Run tests with coverage report
npm run test:live        # Run live network tests (hits real APIs)
npm run deps:audit       # Dependency audit (unused/unlisted packages)

# Run a single test file
npx vitest run path/to/test.ts

# Run tests matching a pattern
npx vitest run -t "test name pattern"

# Run tests in a specific directory
npx vitest run src/lib/server/monitoring
```

## Code Style

### Formatting (Prettier)

- **Tabs** for indentation (not spaces)
- **Single quotes** for strings
- **No trailing commas**
- **Print width**: 100 characters
- Plugins: `prettier-plugin-svelte`, `prettier-plugin-tailwindcss`

Run `npm run format` before committing.

### Imports

```typescript
// 1. External packages (Node built-ins use node: prefix)
import { randomUUID } from 'node:crypto';
import { json } from '@sveltejs/kit';
import { z } from 'zod';

// 2. Internal imports using $lib alias
import { logger } from '$lib/logging';
import { ValidationError } from '$lib/errors';
import type { MovieContext } from '$lib/server/monitoring/specifications/types';

// 3. Relative imports (same directory)
import { reject, accept } from './utils.js';
```

Always include `.js` extension in imports for ES modules.

### TypeScript

- **Strict mode** is enabled
- Use **Zod schemas** (`src/lib/validation/schemas.ts`) for runtime validation
- Derive types from Drizzle schema using `$inferSelect` and `$inferInsert`:

```typescript
export type MovieRecord = typeof movies.$inferSelect;
export type NewMovieRecord = typeof movies.$inferInsert;
```

### Naming Conventions

| Type                     | Convention           | Example                                         |
| ------------------------ | -------------------- | ----------------------------------------------- |
| Variables/functions      | camelCase            | `movieCount`, `getMovieById`                    |
| Classes/interfaces/types | PascalCase           | `MovieUpgradeableSpecification`, `MovieContext` |
| Constants                | SCREAMING_SNAKE_CASE | `CURRENT_SCHEMA_VERSION`, `SECURITY_HEADERS`    |
| Files                    | kebab-case           | `upgradeable-specification.ts`                  |
| Svelte components        | PascalCase           | `IndexerModal.svelte`                           |
| Database tables          | camelCase (plural)   | `movies`, `episodeFiles`                        |

## Error Handling

Use the `AppError` hierarchy from `$lib/errors`:

```typescript
import { ValidationError, NotFoundError, ExternalServiceError, isAppError } from '$lib/errors';

// Throw typed errors
throw new ValidationError('Invalid input', { field: 'name' });
throw new NotFoundError('Movie', movieId);
throw new ExternalServiceError('TMDB', 'Rate limit exceeded', 429);

// Type guard for catch blocks
if (isAppError(error)) {
	return json({ error: error.message, code: error.code }, { status: error.statusCode });
}
```

## Svelte 5 Patterns

### Component Props

```svelte
<script lang="ts">
	interface Props {
		open: boolean;
		data: Movie;
		onSave: (movie: Movie) => void;
		onClose: () => void;
	}

	let { open, data, onSave, onClose }: Props = $props();
</script>
```

### Reactive State

```svelte
<script lang="ts">
	let count = $state(0);
	let name = $state('');

	const doubled = $derived(count * 2);
	const isValid = $derived(name.length > 0);

	$effect(() => {
		console.log('Name changed:', name);
	});
</script>
```

### Modal Form Pattern

Initialize form state with defaults, sync from props in `$effect` when modal opens:

```svelte
<script lang="ts">
	let formData = $state({ name: '', priority: 25 });

	$effect(() => {
		if (open) {
			formData = {
				name: indexer?.name ?? '',
				priority: indexer?.priority ?? 25
			};
		}
	});
</script>
```

## API Routes

### Structure

```typescript
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';
import { indexerCreateSchema } from '$lib/validation/schemas';
import { ValidationError } from '$lib/errors';

export const GET: RequestHandler = async () => {
	const items = await service.getAll();
	return json(items);
};

export const POST: RequestHandler = async ({ request }) => {
	let data: unknown;
	try {
		data = await request.json();
	} catch {
		return json({ error: 'Invalid JSON body' }, { status: 400 });
	}

	const result = indexerCreateSchema.safeParse(data);
	if (!result.success) {
		return json({ error: 'Validation failed', details: result.error.flatten() }, { status: 400 });
	}

	// Use result.data (typed and validated)
	const created = await service.create(result.data);
	return json({ success: true, data: created });
};
```

### Validation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoldyTaint/Cinephage](https://github.com/MoldyTaint/Cinephage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
