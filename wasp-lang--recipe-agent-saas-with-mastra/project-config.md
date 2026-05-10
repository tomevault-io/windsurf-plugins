---
trigger: always_on
description: This document gives a quick rundown on how authentication is configured and used within the Wasp application.
---

# 4. Authentication

This document gives a quick rundown on how authentication is configured and used within the Wasp application.

See the Wasp Auth docs for available methods and complete guides [wasp-overview.mdc](mdc:template/app/.cursor/rules/wasp-overview.mdc)

## Wasp Auth Setup

- Wasp provides built-in authentication with minimal configuration via the Wasp config file. 
- Wasp generates all necessary auth routes, middleware, and UI components based on the configuration.
- Example auth configuration in [main.wasp](mdc:main.wasp):
  ```wasp
  app myApp {
    // ... other config
    auth: {
      // Links Wasp auth to your User model in @schema.prisma
      userEntity: User,
      methods: {
        // Enable username/password login
        usernameAndPassword: {},
        // Enable Google OAuth login
        // Requires setting GOOGLE_CLIENT_ID and GOOGLE_CLIENT_SECRET env vars
        google: {},
        // Enable email/password login with verification
        email: {
          // Set up an email sender (Dummy prints to console)
          // See https://wasp-lang.com/docs/auth/email-auth#email-sending
          fromField: {
            name: "Budgeting Vibe",
            email: "noreply@budgetingvibe.com"
          },
          emailVerification: {
            clientRoute: EmailVerificationRoute
          },
          passwordReset: {
            clientRoute: PasswordResetRoute
          }
        }
      },
      // Route to redirect to if auth fails
      onAuthFailedRedirectTo: "/login",
      // Optional: Route after successful signup/login
      // onAuthSucceededRedirectTo: "/dashboard"
    }
    emailSender: {
      provider: Dummy // Use Dummy for local dev (prints emails to console)
      // provider: SMTP // For production, configure SMTP
    }
  }

  // Define the routes needed by email auth methods
  route EmailVerificationRoute { path: "/auth/verify-email", to: EmailVerificationPage }
  page EmailVerificationPage { component: import { EmailVerification } from "@src/features/auth/EmailVerificationPage.tsx" }

  route PasswordResetRoute { path: "/auth/reset-password", to: PasswordResetPage }
  page PasswordResetPage { component: import { PasswordReset } from "@src/features/auth/PasswordResetPage.tsx" }
  ```

- **Dummy Email Provider Note:** When `emailSender: { provider: Dummy }` is configured in [main.wasp](mdc:main.wasp), Wasp does not send actual emails. Instead, the content of verification/password reset emails, including the clickable link, will be printed directly to the server console where `wasp start` is running.

## Wasp Auth Rules

- **User Model ( [schema.prisma](mdc:schema.prisma) ):**
  - Wasp Auth methods handle essential identity fields (like `email`, `password hash`, `provider IDs`, `isVerified`) internally. These are stored in separate Prisma models managed by Wasp (`AuthProvider`, `AuthProviderData`).
  - Your Prisma `User` model (specified in [main.wasp](mdc:main.wasp) as `auth.userEntity`) typically **only needs the `id` field** for Wasp to link the auth identity.
    ```prisma
    // Minimal User model in @schema.prisma
    model User {
      id Int @id @default(autoincrement())
      // Add other *non-auth* related fields as needed
      // e.g., profile info, preferences, relations to other models
      // profileImageUrl String?
      // timeZone        String? @default("UTC")
    }
    ```
  - **Avoid adding** `email`, `emailVerified`, `password`, `username`, or provider-specific ID fields directly to *your* `User` model in [schema.prisma](mdc:schema.prisma) unless you have very specific customization needs that require overriding Wasp's default behavior and managing these fields manually.
  - If you need frequent access to an identity field like `email` or `username` for *any* user (not just the logged-in one), see the **Recommendation** in the "Wasp Auth User Fields" section below.

- **Auth Pages:**
  - When initially creating Auth pages (Login, Signup), use the pre-built components provided by Wasp for simplicity:
    - `import { LoginForm, SignupForm } from 'wasp/client/auth';`
    - These components work with the configured auth methods in [main.wasp](mdc:main.wasp).
    - You can customize their appearance or build completely custom forms if needed.

- **Protected Routes/Pages:**
  - Use the `useAuth` hook from `wasp/client/auth` to access the current user's data and check authentication status.
  - Redirect or show alternative content if the user is not authenticated.
  ```typescript
  import { useAuth } from 'wasp/client/auth';
  import { Redirect } from 'wasp/client/router'; // Or use Link

  const MyProtectedPage = () => {
    const { data: user, isLoading, error } = useAuth(); // Returns AuthUser | null

    if (isLoading) return <div>Loading...</div>;
    // If error, it likely means the auth session is invalid/expired
    if (error || !user) {
      // Redirect to login page defined in main.wasp (auth.onAuthFailedRedirectTo)
      // Or return <Redirect to="/login" />;
      return <div>Please log in to access this page.</div>;
    }

    // User is authenticated, render the page content
    // Use helpers like getEmail(user) or getUsername(user) if needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wasp-lang/recipe-agent-saas-with-mastra](https://github.com/wasp-lang/recipe-agent-saas-with-mastra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
