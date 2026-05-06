---
trigger: always_on
description: This document provides comprehensive guidelines for AI agents and LLMs working on this project. These instructions ensure consistency, quality, and adherence to project standards.
---

# Agent Instructions for Link Shortener Project

This document provides comprehensive guidelines for AI agents and LLMs working on this project. These instructions ensure consistency, quality, and adherence to project standards.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Coding Standards](#coding-standards)
4. [Documentation](#documentation)

## ⚠️ CRITICAL: Required Reading Before Code Generation

**🚨 MANDATORY REQUIREMENT 🚨**

Before generating ANY code, you MUST:

1. **READ the relevant documentation files** in the `/docs` directory that apply to your task
2. **FOLLOW the patterns and guidelines** specified in those files
3. **DO NOT proceed** with code generation until you have reviewed the applicable documentation

This is **NOT OPTIONAL**. Failure to read the relevant documentation before generating code will result in:

- Incorrect implementations that don't follow project patterns
- Security vulnerabilities from improper authentication/authorization
- Inconsistent UI components that don't match the project's design system
- Wasted time fixing preventable mistakes

### Required Documentation by Topic

Consult these files based on the task at hand:

- **🔐 [Authentication Guidelines](docs/authentication.md)** - **MUST READ** before implementing any authentication, route protection, user data access, or auth-related features
- **🎨 [UI Components Guidelines](docs/ui-components.md)** - **MUST READ** before creating or installing any UI components, forms, buttons, or styled elements

**Workflow:**

1. Identify what you need to implement
2. Read the corresponding documentation file(s) completely
3. Apply the patterns and guidelines from the documentation
4. Only then generate code

## Project Overview

This is a link shortener application built with modern web technologies. The application allows users to:

- Create shortened URLs
- Track link analytics
- Manage their links with authentication
- Share links publicly or privately

**Project Type:** Full-stack Next.js application  
**Database:** PostgreSQL (via Neon Database)  
**ORM:** Drizzle ORM  
**Authentication:** Clerk  
**Styling:** Tailwind CSS v4

## Technology Stack

### Core Technologies

- **Framework:** Next.js 16.0.10 (App Router)
- **Runtime:** React 19.2.1
- **Language:** TypeScript 5.x
- **Database:** PostgreSQL via @neondatabase/serverless
- **ORM:** Drizzle ORM 0.45.1
- **Authentication:** Clerk (@clerk/nextjs 6.36.2)
- **Styling:** Tailwind CSS 4.x

### Development Tools

- **Package Manager:** npm (implied from workspace)
- **Linter:** ESLint 9.x
- **Database Migrations:** Drizzle Kit 0.31.8
- **TypeScript Executor:** tsx 4.21.0

### UI Libraries

- **Icons:** Lucide React 0.561.0
- **Utilities:** clsx, tailwind-merge, class-variance-authority
- **Animations:** tw-animate-css

## Coding Standards

### General Principles

1. **Type Safety First**

   - Always use TypeScript with strict mode enabled
   - Avoid `any` types; use `unknown` or proper type definitions
   - Export and reuse type definitions across files

2. **Component Organization**

   - Use functional components with TypeScript
   - Place reusable components in appropriate directories
   - Keep components focused and single-responsibility

3. **File Naming**

   - Use kebab-case for directories: `link-manager/`
   - Use PascalCase for React components: `LinkCard.tsx`
   - Use camelCase for utilities and hooks: `useLinks.ts`
   - Use kebab-case for regular TypeScript files: `api-client.ts`

4. **Import Organization**

   - Group imports: external packages → internal modules → types → styles
   - Use path aliases (`@/`) defined in tsconfig.json
   - Avoid circular dependencies

5. **Code Style**
   - Use 2-space indentation
   - Use double quotes for strings (match ESLint config)
   - Use semicolons (TypeScript standard)
   - Prefer const over let; avoid var
   - Use arrow functions for callbacks and functional components

### Next.js Specific

1. **App Router Convention**

   - Use Server Components by default
   - Add `"use client"` directive only when needed
   - Follow Next.js 16.x file conventions (page.tsx, layout.tsx, etc.)

2. **Metadata**

   - Export metadata objects for SEO
   - Use generateMetadata for dynamic pages

3. **Performance**

   - Use dynamic imports for large components
   - Implement proper loading states
   - Optimize images with next/image

4. **⚠️ CRITICAL: Routing & Middleware**
   - **NEVER use middleware.ts** - This is deprecated in Next.js 16.x
   - **ALWAYS use proxy.ts** for request handling and routing logic
   - The project uses proxy.ts as the modern replacement for middleware functionality

### Database & ORM

1. **Drizzle ORM Patterns**

   - Define schemas in `db/schema.ts`
   - Use Drizzle's type-safe query builder
   - Export types from schema definitions
   - Use transactions for multi-step operations

2. **Migrations**
   - Generate migrations with `drizzle-kit`
   - Review generated SQL before applying
   - Never edit schema files directly in production

### Authentication

1. **Clerk Integration**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomphill/linkshortenerproject](https://github.com/tomphill/linkshortenerproject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
