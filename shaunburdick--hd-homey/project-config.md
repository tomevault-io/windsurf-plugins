---
trigger: always_on
description: This document provides AI agents with essential context to quickly understand and work with the HD Homey project.
---

# AI Agent Guide for HD Homey

This document provides AI agents with essential context to quickly understand and work with the HD Homey project.

## Project Overview

**HD Homey** is a Next.js-based proxy application for HDHomeRun devices that enables secure remote access to live TV streams over the internet.

### Tech Stack
- **Framework**: Next.js 16.0.3 (App Router)
- **UI Library**: React 19.2.0
- **Language**: TypeScript 5
- **Database**: SQLite via better-sqlite3 + Drizzle ORM
- **Authentication**: Better-Auth 1.1.0 (username plugin)
- **UI**: new.css for styling
- **Testing**: Vitest + React Testing Library
- **Deployment**: Docker + Docker Compose

## Architecture

### Directory Structure
```
apps/
├── web/               # Main Next.js application
│   ├── migrations/           # Database migrations
│   ├── src/
│   │   ├── app/               # Next.js App Router pages and API routes
│   │   │   ├── (protected)/  # Auth-protected routes (tuners, channels, users, settings)
│   │   │   ├── api/          # API endpoints
│   │   │   └── users/        # Public auth routes (signin, get-started)
│   │   ├── components/       # React components
│   │   ├── lib/              # Utilities and core logic
│   │   │   ├── auth/         # Better-Auth configuration and helpers
│   │   │   │   ├── auth.ts   # Better-Auth instance
│   │   │   │   ├── auth-client.ts  # Client-side auth hooks
│   │   │   │   └── helpers.ts # Role checking (requireAdmin, etc.)
│   │   │   ├── database/     # Database schema and operations
│   │   │   │   └── schema.ts # Unified schema (includes Better-Auth tables)
│   │   │   └── logger.ts     # Pino logging
│   │   └── proxy.ts          # Route protection proxy
│   ├── public/        # Static assets
│   ├── scripts/       # Build scripts
│   └── package.json   # Web app dependencies
├── docs/              # VitePress documentation site
│   ├── .vitepress/    # VitePress configuration
│   ├── getting-started/
│   ├── features/
│   └── package.json   # Docs dependencies
└── android/           # Android app (Phase 1, coming soon)

.specify/            # Spec-kit: specifications and constitution
specs/               # Spec-kit: implementation plans (created during planning phase)
```

### Key Features (with Specs)
1. **Tuner Management** (SPEC-001) - Add/edit/manage HDHomeRun devices
2. **Channel Discovery** (SPEC-002) - Automatic channel lineup scanning and updates
3. **User Authentication** (SPEC-003) - Role-based access (admin/viewer)
4. **User Management** (SPEC-004) - CRUD operations for user accounts
5. **Stream Proxying** - Transparent video stream relay with URL rewriting

## Development Practices

### Spec-Driven Development (Spec-Kit)
- **All features must have a spec** in `.specify/features/`
- Use spec-kit commands: `/speckit.specify`, `/speckit.clarify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.implement`
- Feature specs are single files (e.g., `001-tuner-management.md`), not subdirectories
- Implementation plans live in `specs/###-feature/` directory (created during planning phase)
- Update spec status as implementation progresses
- Specs drive implementation, not vice versa

### Documentation Maintenance

**CRITICAL**: HD Homey uses a dual-documentation approach that MUST be kept in sync:

#### Documentation Structure
1. **README.md** (root) - Quick pitch and getting started (~95 lines)
   - Project description and badges
   - Repository structure overview
   - Minimal quick start guide
   - Feature highlights with links to docs
   - Links to comprehensive documentation
   
2. **apps/docs/** (VitePress site) - Comprehensive documentation (~4,400 lines)
   - Deployed to: https://shaunburdick.github.io/hd-homey/
   - Full installation guides
   - Detailed feature documentation
   - Configuration reference
   - Troubleshooting guides
   - API reference
   - Contributing guide

#### When to Update Documentation

**Always update documentation when you**:
- ✅ Add a new feature or capability
- ✅ Change existing behavior or configuration
- ✅ Add/modify environment variables
- ✅ Update dependencies that affect usage
- ✅ Fix bugs that were documented as workarounds
- ✅ Change API endpoints or contracts
- ✅ Add new troubleshooting solutions

#### Documentation Update Checklist

When making changes, update BOTH locations as needed:

**1. README.md Updates Required When**:
- [ ] Changing the quick start process
- [ ] Adding high-level features to the feature list
- [ ] Updating tech stack versions (major versions only)
- [ ] Changing project status or version

**2. apps/docs/ Updates Required When**:
- [ ] Adding new features → Update `apps/docs/features/`
- [ ] Changing configuration → Update `apps/docs/config/`
- [ ] Adding installation methods → Update `apps/docs/getting-started/installation.md`
- [ ] Fixing common issues → Update `apps/docs/troubleshooting/`
- [ ] Changing APIs → Update `apps/docs/api/`
- [ ] Modifying workflows → Update `apps/docs/contributing/`

**3. Specific Files to Check**:
```
Feature changes:
  → apps/docs/features/[feature-name].md
  → apps/docs/features/index.md (overview)
  → README.md (feature list if major)

Configuration changes:
  → apps/docs/config/environment-variables.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shaunburdick/hd-homey](https://github.com/shaunburdick/hd-homey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
