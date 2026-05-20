---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# Payload App Starter - AI Assistant Guide

## Project Overview

This is a modern SaaS starter kit built with Next.js 15 and Payload CMS, designed to accelerate SaaS development. The project uses TypeScript, PostgreSQL, and includes a complete authentication system with role-based access control.

## Tech Stack

- **Framework**: Next.js 15.5.3 with App Router
- **CMS**: Payload CMS 3.55.1
- **Database**: PostgreSQL with Payload adapter
- **Language**: TypeScript 5.7.3
- **Styling**: Tailwind CSS v4 with Shadcn UI components
- **Storage**: Vercel Blob Storage (with optional Cloudflare R2/AWS S3 support)
- **Node**: v18.20.2+ or v20.9.0+
- **Package Manager**: pnpm

## Architecture Overview

### Core Architecture Patterns
- **App Router Architecture**: Uses Next.js 15 App Router with clear separation between frontend and Payload admin routes
- **Server-First Approach**: Defaults to Server Components, using Client Components only when necessary for interactivity
- **Type Safety**: Leverages Payload's automatic type generation for end-to-end type safety
- **Authentication Flow**: Cookie-based auth with middleware protection, server actions for mutations
- **Storage Abstraction**: Configurable storage backend (Vercel Blob/S3/R2) through Payload plugins

### Route Organization
- Public routes: `/(site)/*` - Accessible to all users
- Auth routes: `/(auth)/*` - Login, register, password reset (redirects if already authenticated)
- Protected routes: `/(admin)/*` - Requires authentication via middleware
- Payload admin: `/(payload)/*` - CMS admin interface
- API routes: `/api/*` - REST endpoints, Payload API at `/api`, GraphQL at `/api/graphql`

## Project Structure

```
/src
  /app                 # Next.js App Router
    /(frontend)        # Frontend routes
      /(admin)         # Protected admin routes (requires authentication)
      /(auth)          # Authentication routes (login, register)
      /(site)          # Public site routes
    /(payload)         # Payload CMS routes
    /api               # API routes including auth verification
  /collections         # Payload collections (Users, Media)
  /components          # React components
    /auth              # Authentication components
    /dashboard         # Dashboard components
    /ds.tsx            # Design system exports
    /site              # Site components (header, footer)
    /theme             # Theme components (dark/light mode)
    /ui                # Shadcn UI components
  /lib                 # Utility functions
    /auth.ts           # Authentication utilities and server actions
    /email.ts          # Email service with Resend
    /validation.ts     # Form validation schemas
  /middleware.ts       # Next.js middleware for route protection
  /payload.config.ts   # Payload CMS configuration
  /payload-types.ts    # Auto-generated Payload types
```

## Development Commands

```bash
# Install dependencies
pnpm install

# Development server
pnpm dev

# Safe development (clears .next cache)
pnpm devsafe

# Build for production
pnpm build

# Start production server
pnpm start

# Generate Payload import map
pnpm generate:importmap

# Generate TypeScript types
pnpm generate:types

# Run linter
pnpm lint

# Access Payload CLI
pnpm payload

# Testing
# Note: No test framework is currently configured.
# To run tests, first install a test framework (e.g., Jest, Vitest)
```

## Environment Variables

Required environment variables (see `.env.example`):

```bash
# Database
DATABASE_URI=postgres://postgres:<password>@127.0.0.1:5432/your-database-name

# Payload secret key
PAYLOAD_SECRET=YOUR_SECRET_HERE

# Email Configuration (Resend)
RESEND_API_KEY=re_xxxxxxxx
EMAIL_FROM=noreply@yourdomain.com

# Storage
BLOB_READ_WRITE_TOKEN=YOUR_READ_WRITE_TOKEN_HERE

# Optional: Cloudflare R2
R2_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID_HERE
R2_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY_HERE
R2_BUCKET=YOUR_BUCKET_HERE
R2_ENDPOINT=YOUR_ENDPOINT_HERE
```

Note: The example shows minimal required variables. Additional optional variables may be needed for S3/R2 storage.

## Important Files

### Configuration
- `/src/payload.config.ts` - Payload CMS configuration
- `/next.config.mjs` - Next.js configuration with security headers
- `/tsconfig.json` - TypeScript configuration with path aliases
- `/postcss.config.mjs` - PostCSS configuration for Tailwind CSS v4
- `/Dockerfile` - Docker configuration for containerized deployment

### Collections
- `/src/collections/Users.ts` - User collection with authentication fields
- `/src/collections/Media.ts` - Media/file upload collection

### Authentication System
- `/src/lib/auth.ts` - Core authentication utilities and server actions
- `/src/lib/email.ts` - Email service with Resend integration
- `/src/middleware.ts` - Route protection middleware
- `/src/components/auth/` - Authentication UI components
  - `login-form.tsx` - Login form with toast notifications
  - `register-form.tsx` - Registration form with email verification
  - `logout-button.tsx` - Client-side logout with router navigation
  - `logout-form.tsx` - Server action logout form (works without JS)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brijr/payload-starter](https://github.com/brijr/payload-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
