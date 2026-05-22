---
trigger: always_on
description: Guidelines for creating and maintaining Cursor rules to ensure consistency and effectiveness.
---

- **Required Rule Structure:**
  ```markdown
  ---
  description: Clear, one-line description of what the rule enforces
  globs: path/to/files/*.ext, other/path/**/*
  alwaysApply: boolean
  ---

  - **Main Points in Bold**
    - Sub-points with details
    - Examples and explanations
  ```

- **File References:**
  - Use `[filename](mdc:path/to/file)` ([filename](mdc:filename)) to reference files
  - Example: [prisma.mdc](mdc:.cursor/rules/prisma.mdc) for rule references
  - Example: [schema.prisma](mdc:prisma/schema.prisma) for code references

- **Code Examples:**
  - Use language-specific code blocks
  ```typescript
  // ✅ DO: Show good examples
  const goodExample = true;

  // ❌ DON'T: Show anti-patterns
  const badExample = false;
  ```

- **Rule Content Guidelines:**
  - Start with high-level overview
  - Include specific, actionable requirements
  - Show examples of correct implementation
  - Reference existing code when possible
  - Keep rules DRY by referencing other rules

- **Rule Maintenance:**
  - Update rules when new patterns emerge
  - Add examples from actual codebase
  - Remove outdated patterns
  - Cross-reference related rules

- **Best Practices:**
  - Use bullet points for clarity
  - Keep descriptions concise
  - Include both DO and DON'T examples
  - Reference actual code over theoretical examples
  - Use consistent formatting across rules

- **Environment Variable Loading (Production & Amplify/AWS Lambda Ready)**
  - Lazily load environment variables inside functions, not at module top-level
  - Use a helper function to check for multiple prefixes (e.g., `AMPLIFY_`, `AWS_AMPLIFY_`, and no prefix)
  - Support both server and client contexts (Next.js config and `process.env`)
  - Example: See `getStripeEnvVar` in [`src/lib/stripe/init.ts`](mdc:src/lib/stripe/init.ts)

- **Next.js Environment Variables in Production (AWS Amplify)**
  - **CRITICAL**: All environment variables must be declared in the `env` section of [`next.config.mjs`](mdc:next.config.mjs) to be available at runtime in production
  - Environment variables set in AWS Amplify console are not automatically available unless explicitly declared in Next.js config
  - **AWS Amplify Environment Variable Pattern**: AWS Amplify prefixes environment variables with `AMPLIFY_` in the runtime, even if you set them without the prefix in the console
  - **NEXT_PUBLIC_ Variables in AWS Amplify**: `NEXT_PUBLIC_` prefixed variables may not be available in server-side contexts (API routes, server components) in AWS Amplify production environment
  - ✅ DO: Use `AMPLIFY_` prefixed variables for AWS Amplify deployments with fallbacks for local development
  ```javascript
  // next.config.mjs
  env: {
    // API JWT credentials - prioritize AMPLIFY_ prefix for AWS Amplify
    API_JWT_USER: process.env.AMPLIFY_API_JWT_USER || process.env.API_JWT_USER,
    API_JWT_PASS: process.env.AMPLIFY_API_JWT_PASS || process.env.API_JWT_PASS,
    AMPLIFY_API_JWT_USER: process.env.AMPLIFY_API_JWT_USER,
    AMPLIFY_API_JWT_PASS: process.env.AMPLIFY_API_JWT_PASS,
    // Keep fallbacks for local development
    NEXT_PUBLIC_API_JWT_USER: process.env.NEXT_PUBLIC_API_JWT_USER,
    NEXT_PUBLIC_API_JWT_PASS: process.env.NEXT_PUBLIC_API_JWT_PASS,
  }
  ```
  ```typescript
  // Helper functions should prioritize AMPLIFY_ prefix
  export function getApiJwtUser() {
    return (
      process.env.AMPLIFY_API_JWT_USER ||
      process.env.API_JWT_USER ||
      process.env.NEXT_PUBLIC_API_JWT_USER
    );
  }
  ```
  - ❌ DON'T: Rely only on `NEXT_PUBLIC_` prefixed variables for server-side authentication in AWS Amplify
  - ❌ DON'T: Only set environment variables in AWS Amplify console without declaring them in Next.js config
  - **Rationale**: AWS Amplify's runtime environment behavior differs from standard Next.js deployments. `AMPLIFY_` prefixed variables are reliable in production while `NEXT_PUBLIC_` variables may be unavailable in server contexts
  - **Debugging**: Variables will show as `UNDEFINED` in production logs if not declared in config or using wrong prefix pattern

- **DTO (Data Transfer Object) Setup and Usage**
  - **Centralize DTO Definitions**
    - Define all DTOs in `src/types/index.ts` (or submodules if the file grows large)
    - Use TypeScript `interface` or `type` for DTOs, matching backend API schema
    ```typescript
    // ✅ DO: Centralize DTOs
    export interface UserProfileDTO {
      id: number | null;
      userId: string;
      firstName: string;
      lastName: string;
      email: string;
      // ...other fields
    }
    ```
  - **Keep DTOs Flat and Serializable**
    - Avoid methods or computed properties; use only serializable types
    ```typescript
    // ✅ DO: Use only serializable fields
    export interface SubscriptionDTO {
      id: number | null;
      userId: string;
      plan: string;
      status: 'active' | 'inactive' | 'canceled';
      // ...other fields
    }
    ```
  - **Match Backend Schema**
    - Align DTO fields/types with backend API (OpenAPI/Swagger, Prisma, REST docs)
    - Document intentional differences
    ```typescript
    // ✅ DO: Match backend schema
    export interface EventDTO {
      id: number;
      name: string;
      date: string; // ISO string

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
