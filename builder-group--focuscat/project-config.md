---
trigger: always_on
description: TypeScript coding standards and style guidelines
---


# TypeScript Style Guide

TypeScript coding standards and style guidelines for our codebase. These guidelines ensure consistency, maintainability, and high code quality across TypeScript projects.

## Core Principles

- **KISS (Keep It Simple, Stupid)** - Always choose the simplest, most maintainable solution
- **TypeScript First** - Always use TypeScript with strict typing (`strict: true`) everywhere
- **Less is More** - Always avoid unnecessary complexity, the best code is no code
- **Self-Documenting** - Always make code obvious and clear without comments

## File Organization

### Directory Structure

- Always organize code in a predictable and scalable way
- Always keep related code close together
- Always use clear, descriptive directory names
- Always follow consistent patterns across the project
- Always use singular for categories/domains (e.g. `auth/`, `user/`, `product/`)
- Always use plural for collections/lists (e.g. `components/`, `hooks/`, `utils/`)

✅ Good:

```typescript
src/
  auth/              # Singular: domain
    components/      # Plural: collection
    hooks/          # Plural: collection
    lib/            # Singular: category

  user/             # Singular: domain
    components/     # Plural: collection
    hooks/         # Plural: collection
    lib/           # Singular: category

  lib/             # Singular: core category
  components/      # Plural: shared collection
  hooks/          # Plural: shared collection
```

❌ Bad:

```typescript
src/
  auths/           # Wrong: Category should be singular
    component/     # Wrong: Collection should be plural

  users/          # Wrong: Category should be singular
    hook/         # Wrong: Collection should be plural

  libraries/      # Wrong: Category should be singular
  shared-components/ # Wrong: Use simple plural
```

### Files & Directories

- Always use consistent and predictable naming patterns
- Always make names descriptive and purpose-indicating
- Always follow established community conventions

✅ Good:

```typescript
// Directories (kebab-case)
src/
  auth/
  components/
  hooks/
  lib/

// Regular Files (kebab-case)
user-service.ts
jwt-utils.ts
date-formatter.ts
api-client.ts

// Component Files (PascalCase)
UserProfileCard.tsx
OrderSummaryTable.tsx
PaymentMethodSelector.tsx
ButtonPrimary.tsx

// Class Files (PascalCase)
OrderProcessor.ts
PaymentGateway.ts
CacheManager.ts
```

❌ Bad:

```typescript
// Directories (mixed case)
src/
  UserManagement/     # Wrong: PascalCase directory
  order_processing/   # Wrong: snake_case directory
  PAYMENT/           # Wrong: UPPERCASE directory
  Shared-Utils/      # Wrong: Mixed kebab-case and PascalCase

// Files (inconsistent)
userService.ts      # Wrong: camelCase
USER_HELPERS.ts    # Wrong: SNAKE_CASE
payment.utilities.ts # Wrong: dot notation
Api.Client.ts      # Wrong: PascalCase with dots
```

### Code Identifiers

- Always use clear, descriptive names that indicate purpose
- Always follow TypeScript community standards
- Always maintain consistent prefixing for special types

✅ Good:

```typescript
// Variables & Functions (camelCase)
const currentUser = getCurrentUser();
const isValidEmail = validateEmail(email);
function calculateTotalPrice(items: TOrderItem[]): number {
	return items.reduce((sum, item) => sum + item.price, 0);
}

// Interfaces (T prefix) - Always use for object shapes
interface TUser {
	id: string;
	email: string;
	profile: TUserProfile;
}

interface TOrderItem {
	id: string;
	productId: string;
	quantity: number;
	price: number;
}

// Types (T prefix) - Only use when interface is not possible
type TOrderStatus = 'pending' | 'processing' | 'completed'; // Union
type TUserOrNull = TUser | null; // Union with null
type TPartialUser = Partial<TUser>; // Mapped type
type TUserConfig = Required<TUserOptions>; // Utility type

// Enums (E prefix)
enum EOrderStatus {
	Pending = 'pending',
	Processing = 'processing',
	Completed = 'completed',
	Cancelled = 'cancelled'
}

enum EUserRole {
	Admin = 'admin',
	Customer = 'customer',
	Guest = 'guest'
}

// Schemas (S prefix)
const SUserProfile = z.object({
	firstName: z.string().min(2),
	lastName: z.string().min(2),
	dateOfBirth: z.iso.datetime().optional(),
	phoneNumber: z
		.string()
		.regex(/^\+?[1-9]\d{1,14}$/)
		.optional()
});

const SOrderCreate = z.object({
	userId: z.string().uuid(),
	items: z.array(
		z.object({
			productId: z.string().uuid(),
			quantity: z.number().int().positive()
		})
	)
});
```

❌ Bad:

```typescript
// Variables & Functions (inconsistent)
const CurrentUser = getCurrentUser();  // Wrong: PascalCase
const valid_email = validate_email();  // Wrong: snake_case
const CALCULATE_PRICE = () => {};      // Wrong: UPPER_CASE

// Types & Interfaces (missing prefix or wrong keyword)
type User = {                         // Wrong: Missing T prefix
  ID: string;                        // Wrong: UPPER_CASE
  Email: string;                     // Wrong: PascalCase
};

type TOrderItem = {                   // Wrong: Use interface for objects
  product_id: string;               // Wrong: snake_case
  Quantity: number;                 // Wrong: PascalCase
};

interface TUserStatus = 'active' | 'inactive'; // Wrong: Can't use interface for unions

// Enums (inconsistent)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [builder-group/focuscat](https://github.com/builder-group/focuscat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
