---
trigger: always_on
description: Defines the directory and file organization for the Kostra project. Ensures consistency and maintainability across the codebase. Make sure any file you create follows the structure below. Make sure you don't put components outside of this directory structure unless they don't fit
---


# Project Structure

**Description:**
Defines the directory and file organization for the Kostra project. Ensures consistency and maintainability across the codebase. Make sure any file you create follows the structure below. Make sure you don't put components outside of this directory structure unless they don't fit

**Auto Attachments:**

- Applies to all files and folders in the repository.

The project follows a well-organized directory structure:

```
sass-boilerplate/
├── docs/                      # Project documentation
│   ├── error-handling-README.md
│   └── file-upload-README.md
│
├── infra/                     # Infrastructure and deployment configs
│   ├── docker-compose.yml
│   ├── Dockerfile
│   └── Dockerfile-local.dockerfile
│
├── prisma/                    # Database schema and migrations
│   ├── migrations/           # Database migration files
│   └── schema.prisma         # Prisma schema definition
│
├── public/                    # Static assets
│   ├── favicon/              # Favicon assets
│   ├── logos/                # Brand logo files
│   └── screenshots/          # Application screenshots
│
├── src/
│   ├── app/                   # Next.js app router
│   │   ├── (branding)/       # Landing page and marketing routes
│   │   │   ├── contact/
│   │   │   ├── privacy-policy/
│   │   │   └── terms-of-services/
│   │   │
│   │   ├── api/              # API routes and endpoints
│   │   │   ├── admin/        # Admin management endpoints
│   │   │   ├── auth/         # Authentication endpoints
│   │   │   ├── billing/      # Billing and payment endpoints
│   │   │   ├── blogs/        # Blog management endpoints
│   │   │   ├── categories/   # Category management endpoints
│   │   │   ├── contact/      # Contact form endpoints
│   │   │   ├── file-upload/  # File upload endpoints
│   │   │   ├── files/        # File management endpoints
│   │   │   ├── packages/     # Package management endpoints
│   │   │   ├── users/        # User management endpoints
│   │   │   └── webhooks/     # Webhook handlers
│   │   │
│   │   ├── app/              # Protected application routes
│   │   │   ├── admin/        # Admin dashboard pages
│   │   │   ├── blogs/        # Blog management pages
│   │   │   ├── categories/   # Category management pages
│   │   │   ├── contact-management/
│   │   │   ├── credit-history/
│   │   │   ├── details/
│   │   │   ├── files/        # File management pages
│   │   │   ├── packages/     # Package management pages
│   │   │   └── settings/     # User settings pages
│   │   │
│   │   ├── auth/             # Authentication related pages
│   │   │   └── signin/
│   │   │
│   │   └── onboarding/       # User onboarding flow
│   │
│   ├── components/            # Reusable UI components
│   │   ├── atom/             # Basic building blocks (buttons, inputs, etc.)
│   │   ├── molecules/        # Combinations of atoms
│   │   │   ├── blogs/
│   │   │   ├── common/
│   │   │   ├── editor/
│   │   │   ├── files/
│   │   │   └── form/
│   │   ├── organisms/        # Complex UI components
│   │   │   ├── modules/
│   │   │   └── shared/
│   │   ├── icons/            # SVG icons and icon components
│   │   └── branding/         # Landing page components
│   │
│   ├── data/                 # Static data and constants
│   │   ├── config/          # Configuration data
│   │   ├── data.ts          # Static data exports
│   │   └── schema.ts        # Data schemas
│   │
│   ├── hooks/                # Custom React hooks
│   │   ├── useAdminUsers.ts
│   │   ├── useApiMutation.ts
│   │   ├── useBlogs.ts
│   │   ├── useCategories.ts
│   │   ├── useContacts.ts
│   │   ├── useCredits.ts
│   │   ├── useEmailSignIn.ts
│   │   ├── useFiles.ts
│   │   ├── useFileUpload.ts
│   │   ├── useGoogleSignIn.ts
│   │   ├── usePackages.ts
│   │   ├── useSignInModal.ts
│   │   └── useStripe.ts
│   │
│   ├── lib/                  # Utility functions and shared logic
│   │   ├── auth/            # Authentication utilities
│   │   │   └── jwt.ts
│   │   ├── constants/       # Application constants
│   │   ├── email/           # Email service configuration
│   │   │   ├── drivers/     # Email provider drivers
│   │   │   ├── templates/   # Email templates
│   │   │   ├── factory.ts
│   │   │   └── types.ts
│   │   ├── prisma/          # Database client and utilities
│   │   ├── routes/          # Route definitions and utilities
│   │   └── utils/           # Helper functions
│   │
│   ├── providers/            # Context providers and wrappers
│   │   ├── AuthProvider.tsx
│   │   ├── Providers.tsx
│   │   └── SignInModalProvider.tsx
│   │
│   ├── schemas/              # Validation schemas (Zod)
│   │   ├── auth.schema.ts
│   │   ├── blog.schema.ts
│   │   ├── category.schema.ts
│   │   ├── contact.schema.ts
│   │   └── package.schema.ts
│   │
│   ├── services/             # API services and integrations
│   │   ├── api/             # Frontend API client methods
│   │   ├── external/        # Third-party service integrations
│   │   ├── internal/        # Backend business logic services

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [advantailabs/kostra-boilerplate](https://github.com/advantailabs/kostra-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
