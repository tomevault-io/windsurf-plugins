---
trigger: always_on
description: Let's define the project directory conventions for our Next.js and MongoDB specialist agent. This will ensure consistency, maintainability, and clear separation of concerns, which are vital in production environments.
---

Let's define the project directory conventions for our Next.js and MongoDB specialist agent. This will ensure consistency, maintainability, and clear separation of concerns, which are vital in production environments.

---

**Project Directory Conventions:**

The agent will understand and adhere to the following project structure and conventions for a typical Next.js application using the **App Router** and MongoDB:

```
/my-next-app
├── public/                 # Static assets (images, fonts)
│   ├── images/
│   └── favicon.ico
├── src/                    # All application source code
│   ├── app/                # Next.js App Router root
│   │   ├── (auth)/         # Route group for authentication pages (login, register, forgot-password)
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/
│   │   │       └── page.tsx
│   │   ├── api/            # API Routes (Serverless Functions)
│   │   │   ├── auth/       # API routes for authentication (e.g., /api/auth/login)
│   │   │   │   └── route.ts
│   │   │   ├── users/      # API routes for user management
│   │   │   │   └── route.ts
│   │   │   └── posts/
│   │   │       └── route.ts
│   │   ├── [entity]/       # Dynamic route segments (e.g., /products/[slug])
│   │   │   └── page.tsx
│   │   ├── layout.tsx      # Root layout for the entire application
│   │   ├── page.tsx        # Root page (e.g., homepage)
│   │   └── globals.css     # Global styles
│   ├── components/         # Reusable UI components
│   │   ├── ui/             # Generic, unstyled UI components (e.g., Button, Modal) - often from a UI library
│   │   │   └── Button.tsx
│   │   ├── specific/       # Application-specific components (e.g., UserProfileCard, PostList)
│   │   │   └── PostCard.tsx
│   │   └── shared/         # Components shared across different contexts
│   │       └── Navbar.tsx
│   ├── lib/                # Backend utilities, helpers, and database logic
│   │   ├── db/             # Database connection and models
│   │   │   ├── connect.ts      # MongoDB connection utility
│   │   │   ├── models/         # Mongoose models
│   │   │   │   ├── User.ts
│   │   │   │   └── Post.ts
│   │   │   └── utils.ts        # Database-specific helper functions
│   │   ├── auth.ts         # Authentication related utilities (e.g., NextAuth.js config, session helpers)
│   │   ├── utils.ts        # General utility functions (e.g., formatters, validators)
│   │   └── constants.ts    # Application-wide constants
│   ├── hooks/              # Custom React Hooks
│   │   └── useAuth.ts
│   ├── types/              # TypeScript declaration files (interfaces, types)
│   │   └── index.d.ts      # Centralized type definitions
│   └── services/           # Business logic, data fetching functions (client-side/reusable server-side)
│       ├── userService.ts
│       └── postService.ts
├── tests/                  # Unit and integration tests
├── .env.local              # Environment variables
├── next.config.mjs         # Next.js configuration
├── package.json            # Project dependencies and scripts
├── tsconfig.json           # TypeScript configuration
└── tailwind.config.ts      # Tailwind CSS configuration (if used)
```

**Key Conventions and Rationales:**

1.  **`src/` Directory:** All application source code lives here. This is a common practice in larger projects for better organization.
    *   **Rationale:** Clearly separates application code from configuration files and static assets.
2.  **`app/` (Next.js App Router):** Follows Next.js's official App Router structure.
    *   **Route Groups `(folder-name)/`:** Used for organizing routes without affecting the URL path (e.g., `(auth)` for login/register).
    *   **`api/`:** Dedicated directory for API routes. `route.ts` is the standard for App Router API handlers.
    *   **Dynamic Routes `[slug]`:** Standard Next.js convention for dynamic segments.
    *   **`layout.tsx`, `page.tsx`:** Standard App Router files.
    *   **`globals.css`:** Centralized global styles, often imported into the root `layout.tsx`.
3.  **`components/`:**
    *   **`ui/`:** For highly reusable, often unstyled (or styled via a framework like Tailwind) components that are generic in nature (e.g., Button, Input, Modal). These could also come from a headless UI library.
    *   **`specific/`:** For components that are more tightly coupled to specific features or data structures (e.g., `UserProfileCard`, `ProductDetails`).
    *   **`shared/`:** For components like `Navbar`, `Footer`, `Sidebar` that are used across multiple layouts or major sections.
    *   **Rationale:** Promotes reusability, modularity, and makes it easier to locate specific types of components.
4.  **`lib/`:** This is a crucial directory for backend/server-side logic, database interactions, and core utilities.
    *   **`lib/db/`:**
        *   **`connect.ts`:** Handles the single, reusable MongoDB connection.
        *   **`models/`:** All Mongoose schema definitions go here. Each model in its own file.
        *   **`utils.ts`:** Any database-specific helper functions (e.g., `populateUserWithPosts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mdresch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
