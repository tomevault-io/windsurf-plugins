---
trigger: always_on
description: Comprehensive Testing Best Practices for Svelte 5 + vitest-browser-svelte
---


# Testing Rules & Best Practices

You are an expert in Svelte 5, SvelteKit, TypeScript, and modern testing with vitest-browser-svelte.

## Core Testing Principles

### Foundation First Approach
- **Aim for 100% test coverage** using complete test structure planning
- Start with all describe blocks and test stubs using `.skip`
- Implement tests incrementally - remove `.skip` as you write each test
- Test all code paths - every branch, condition, and edge case

### Test File Organization
- **Component Tests**: `*.svelte.test.ts` - Real browser testing
- **SSR Tests**: `*.ssr.test.ts` - Server-side rendering validation
- **Server Tests**: `*.test.ts` - API routes, utilities, business logic

## CRITICAL: vitest-browser-svelte Patterns

### Always Use Locators, Never Containers

```typescript
// ❌ NEVER use containers
const { container } = render(MyComponent);

// ✅ ALWAYS use locators with auto-retry
render(MyComponent);
const button = page.getByTestId('submit');
await button.click();
```

### Handle Strict Mode Violations

```typescript
// ❌ FAILS: Multiple elements match
page.getByRole('link', { name: 'Home' });

// ✅ CORRECT: Use .first(), .nth(), .last()
page.getByRole('link', { name: 'Home' }).first();
```

### Svelte 5 Runes Testing

```typescript
// ✅ Always use untrack() for $derived
expect(untrack(() => derived_value)).toBe(expected);

// ✅ For getters: get function first, then untrack
const derived_fn = state_object.derived_value;
expect(untrack(() => derived_fn())).toBe(expected);
```

### Form Validation Lifecycle

```typescript
// ✅ Test the full lifecycle: valid → validate → invalid → fix
const form = create_form_state({
	email: { value: '', validation_rules: { required: true } },
});
expect(untrack(() => form.is_form_valid())).toBe(true); // Initially valid
form.validate_all_fields();
expect(untrack(() => form.is_form_valid())).toBe(false); // Now invalid
```

## Client-Server Alignment Strategy

### The Problem

Server unit tests with heavy mocking can pass while production breaks due to client-server mismatches.

### Solution: Real FormData/Request Objects

```typescript
// ❌ BRITTLE: Heavy mocking hides mismatches
const mock_request = {
	formData: vi.fn().mockResolvedValue({
		get: vi.fn().mockReturnValue('test@example.com'),
	}),
};

// ✅ ROBUST: Real FormData objects catch mismatches
const form_data = new FormData();
form_data.append('email', 'test@example.com');
const request = new Request('http://localhost/register', {
	method: 'POST',
	body: form_data,
});

// Only mock external services (database), not data structures
vi.mocked(database.create_user).mockResolvedValue({
	id: '123',
	email: 'test@example.com',
});
```

## Foundation First Test Structure

```typescript
describe('ComponentName', () => {
	describe('Initial Rendering', () => {
		test('should render with default props', async () => {
			// Implemented test
		});

		test.skip('should render with all prop variants', async () => {
			// TODO: Test all type combinations
		});
	});

	describe('User Interactions', () => {
		test.skip('should handle click events', async () => {
			// TODO: Real browser click events
		});
	});

	describe('Edge Cases', () => {
		test.skip('should handle empty data gracefully', async () => {
			// TODO: Test with null/undefined/empty arrays
		});
	});

	describe('Accessibility', () => {
		test.skip('should have proper ARIA roles', async () => {
			// TODO: Test accessibility features
		});
	});
});
```

## SSR Testing Essentials

```typescript
import { render } from 'svelte/server';

test('should render without errors', () => {
	expect(() => render(ComponentName)).not.toThrow();
});

test('should render essential content', () => {
	const { body } = render(ComponentName);
	expect(body).toContain('expected-content');
});
```

## CRITICAL: Avoid Testing Implementation Details

Don't test exact implementation details that provide no user value.

```typescript
// ❌ BRITTLE - Tests exact SVG path data
expect(body).toContain('M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z');

// ✅ ROBUST - Tests semantic styling and structure
expect(body).toContain('text-success');
expect(body).toContain('<svg');

// ✅ BEST - Tests user-visible behavior
await expect.element(page.getByRole('img', { name: /success/i })).toBeInTheDocument();
```

**Why**: SVG paths change when icon libraries update. Test CSS classes, semantic structure, and user experience instead.

## Common Error Solutions

### "strict mode violation: getByRole() resolved to X elements"

- **Cause**: Multiple elements match (common with responsive navigation)
- **Solution**: Use `.first()`, `.nth()`, `.last()` to target specific elements

### "Expected 2 arguments, but got 0"

- **Cause**: Mock function signature doesn't match actual function
- **Solution**: Update mock to accept correct number of arguments

### "lifecycle_outside_component"

- **Cause**: Trying to call `getContext` in test
- **Solution**: Skip the test and add TODO comment for Svelte 5

### Role and Accessibility Confusion

```typescript
// ❌ WRONG: Looking for link when element has role="button"
page.getByRole('link', { name: 'Submit' }); // <a role="button">Submit</a>

// ✅ CORRECT: Use the actual role
page.getByRole('button', { name: 'Submit' });


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techniq/layerchart](https://github.com/techniq/layerchart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
