---
trigger: always_on
description: You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.
---


  You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.
  
  Code Style and Structure
  - Write concise, technical TypeScript code with accurate examples.
  - Use functional and declarative programming patterns; avoid classes.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
  - Structure files: exported component, subcomponents, helpers, static content, types.
  
  Naming Conventions
  - Use lowercase with dashes for directories (e.g., components/auth-wizard).
  - Favor named exports for components.
  
  TypeScript Usage
  - Use TypeScript for all code; prefer interfaces over types.
  - Avoid enums; use maps instead.
  - Use functional components with TypeScript interfaces.
  
  Syntax and Formatting
  - Use the "function" keyword for pure functions.
  - Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
  - Use declarative JSX.
  
  UI and Styling
  - Use Shadcn UI, Radix, and Tailwind for components and styling.
  - Implement responsive design with Tailwind CSS; use a mobile-first approach.
  
  Performance Optimization
  - Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC).
  - Wrap client components in Suspense with fallback.
  - Use dynamic loading for non-critical components.
  
  Key Conventions
  - Use 'nuqs' for URL search parameter state management.
  - Optimize Web Vitals (LCP, CLS, FID).
  - Limit 'use client':
    - Favor server components and Next.js SSR.
    - Use only for Web API access in small components.
    - Avoid for data fetching or state management.
  
  Follow Next.js docs for Data Fetching, Rendering, and Routing.

  Components
  - Ensure each component has a single responsibility to maintain focus and simplicity.
  - Organize components in a structured way with subfolders for components, hooks, and types.
  - Split large components into smaller, reusable ones to enhance modularity.
  - Use error boundaries for larger components to handle errors gracefully.
  - Define props as types and pass them to components for type safety.
  - Extract logic into custom hooks to keep components clean and focused on presentation.
  - Use useMemo and useCallback to optimize performance by preventing unnecessary computations and re-renders.

  Fetching Data
  - Use React Query for data fetching to manage caching, retries, and background refetching efficiently.
  - Implement proper error handling in components using React Query's error states for user feedback.
  - Create reusable hooks for common data-fetching scenarios to reduce code duplication and ensure consistency.

  File Colocation
  - Place components, functions, and types close to where they are used for better accessibility.
  - Each page should have its own folder with related components and functions for modularity.
  - Place shared components or functions at the nearest common ancestor in the directory hierarchy to avoid complexity.

  Functions
  - Name functions clearly to indicate their purpose for better readability.
  - Ensure functions do only one thing, adhering to the Single Responsibility Principle.
  - Place functions in appropriate locations based on their usage (see File Colocation rules).
  - Implement error handling and logging within functions to improve debugging and monitoring.
  - Use descriptive parameter names and group parameters into objects or types when necessary for clarity.

  Logging
  - Use the logger which is located at src/app/functions/log.ts
  - Utilize the provided logging abstraction (e.g., Sentry wrapper) for consistent logging across the codebase.
  - Log important events such as function starts, errors, and user interactions for traceability.
  - Use log.context to provide additional context for logs to aid in debugging.
  - The logger is imported and used like this:
  import { log } from "@/app/functions/log";
  log.info("Something happened", {
    userId: 123,
    userName: "John Doe",
  });
  log.error("Internal Server Error while switching institution", error);

  Mutating Data
  - Create UI components that trigger mutations and display data for user interaction.
  - Use optimistic updates via hooks for immediate UI feedback before server confirmation.
  - Handle validation and database updates on the server side using server actions for security and consistency.
  Code Example: 
  ```ts
"use server";

import { z } from "zod";
import { prisma } from "@/src/app/misc/singletons/prisma";
import { log } from "@/src/app/functions/log";
import { hasPermission } from "@/src/app/functions/server/roles/permission";

// Define validation schema using Zod
const updateCourseSchema = z.object({
  courseId: z.string().min(1, "Course ID is required"),
  name: z
    .string()
    .min(1, "Name cannot be empty")
    .max(100, "Name must be 100 characters or less"),
});

/**
 * Server action to update a course name.
 * Validates input and updates the database securely.
 * @param {string} courseId - The ID of the course to update.
 * @param {string} name - The new name for the course.
 * @returns {Promise<object>} - The updated course object.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AAlber) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
