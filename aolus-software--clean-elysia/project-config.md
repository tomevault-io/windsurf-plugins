---
trigger: always_on
description: This is a Clean Architecture Elysia.js project using Bun runtime with the following tech stack:
---

# GitHub Copilot Instructions for Clean Elysia Project

## Project Overview

This is a Clean Architecture Elysia.js project using Bun runtime with the following tech stack:

- Elysia.js with Bun runtime
- BullMQ for job queues
- Redis for caching and queue backend
- Drizzle ORM for database operations
- PostgreSQL as primary database
- TypeScript for type safety

## Code Style and Conventions

### Comment Style

Comments should be minimal and placed only where necessary to explain complex logic or business rules. Do not add comments every 2-4 lines. Use block comments for entire functions or logical sections.

Good example:

```typescript
/**
 * Validates user credentials and returns authentication token
 * Checks email verification status and account status before allowing login
 */
export const singIn = async (
	email: string,
	password: string,
): Promise<UserInformation> => {
	const user = await UserRepository().findByEmail(email);

	if (!user) {
		throw new BadRequestError("Validation error", [
			{
				field: "email",
				message: "Invalid email or password",
			},
		]);
	}

	const isPasswordValid = await Hash.compareHash(password, user.password);
	return await UserRepository().UserInformation(user.id);
};
```

Bad example:

```typescript
// Find user by email
const user = await UserRepository().findByEmail(email);

// Check if user exists
if (!user) {
	// Throw error
	throw new BadRequestError("Validation error", [
		{
			field: "email",
			message: "Invalid email or password",
		},
	]);
}

// Validate password
const isPasswordValid = await Hash.compareHash(password, user.password);

// Return user information
return await UserRepository().UserInformation(user.id);
```

### Import Organization

Use absolute imports with granular path aliases configured in tsconfig.json. Each directory in `src/libs` has its own dedicated alias:

```typescript
import { BadRequestError, UnauthorizedError } from "@errors";
import { db, users, userRoles } from "@database";
import { UserRepository } from "@repositories";
import { Hash, log } from "@utils";
import { UserInformation, DatatableType } from "@types";
import { StrongPassword } from "@default";
import { AuthPlugin } from "@plugins";
import { eq, and, or } from "drizzle-orm";
```

**Available Path Aliases:**

- `@base` - Base Elysia app configuration
- `@bull` - Queue and worker files
- `@cache` - Cache utilities and constants
- `@config` - Configuration files (AppConfig, DatabaseConfig, etc.)
- `@database` - Database related (db instance, schemas, tables, RedisClient)
- `@default` - Default constants (StrongPassword, paginationLength, etc.)
- `@errors` - Custom error classes
- `@guards` - Authorization guards (RoleGuard, PermissionGuard)
- `@mailer` - Email services and templates
- `@plugins` - Elysia plugins (AuthPlugin, SecurityPlugin, etc.)
- `@repositories` - Repository pattern implementations
- `@types` - TypeScript type definitions and interfaces
- `@utils` - Utility functions (Hash, log, ResponseToolkit, etc.)
- `@modules` - Application modules

**Import Grouping Order:**

1. External libraries (elysia, drizzle-orm, bullmq, etc.)
2. Granular aliases by category:
   - Configuration: `@config`
   - Database: `@database`
   - Errors: `@errors`
   - Types: `@types`
   - Repositories: `@repositories`
   - Utils: `@utils`
   - Others as needed
3. Relative imports (if absolutely necessary)
4. Type-only imports

**Examples:**

```typescript
// Module service example
import { BadRequestError } from "@errors";
import { db, emailVerifications, users } from "@database";
import { ForgotPasswordRepository, UserRepository } from "@repositories";
import { Hash, log } from "@utils";
import { UserInformation } from "@types";
import { AuthMailService } from "@mailer";
import { eq } from "drizzle-orm";

// Module index example
import { AuthPlugin } from "@plugins";
import { JWT_CONFIG } from "@config";
import { CommonResponseSchemas, ResponseToolkit } from "@utils";
import { UserInformation } from "@types";
import Elysia from "elysia";

// Repository example
import { BadRequestError, UnauthorizedError } from "@errors";
import { db, DbTransaction, userRoles, users } from "@database";
import { Hash } from "@utils";
import { defaultSort } from "@default";
import { DatatableType, PaginationResponse, UserInformation } from "@types";
```

### File Naming

- Use kebab-case for file names: `user.repository.ts`, `send-mail-queue.ts`
- Use `.service.ts` suffix for service files
- Use `.repository.ts` suffix for repository files
- Use `.plugin.ts` suffix for Elysia plugins
- Use `.config.ts` suffix for configuration files
- Use `schema.ts` for validation schemas

### Code Structure

#### Repository Pattern

Repositories should return factory functions that provide database access methods:

```typescript
export const UserRepository = () => {
	const dbInstance = db;

	return {
		db: dbInstance,
		getDb: (tx?: DbTransaction) => tx || dbInstance.$cache,

		findByEmail: async (email: string): Promise<User | null> => {
			const result = await dbInstance
				.select()
				.from(users)
				.where(eq(users.email, email))
				.limit(1);
			return result[0] || null;
		},
	};
};
```

#### Service Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aolus-software/clean-elysia](https://github.com/aolus-software/clean-elysia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
