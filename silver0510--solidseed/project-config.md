---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SolidSeed** is a modern CRM platform designed specifically for real estate professionals (realtors, agents, and loan officers). It serves as a replacement for FollowUpBoss, focusing on:

- **Client Hub**: Centralized client management with profiles, documents, notes, tasks, and tag-based organization
- **Email Marketing**: Integrated communication tracking and automation (planned)
- **Mobile-First Design**: Optimized for on-the-go access during property showings
- **Data Portability**: Agents retain their client data when switching brokers
- **SAAS Model**: Subscription tiers (trial, free, pro, enterprise) with 14-day trial period

### Current Status

The project is in early development with two primary features in planning:

1. **User Authentication** - Email/password + OAuth (Google, Microsoft) with Better Auth library and Supabase
2. **Client Hub** - Client management platform with GDPR compliance

## Architecture

### Technology Stack

**Database & Storage:**

- **Supabase** - PostgreSQL database hosting and management
- Supabase CLI for migrations (`supabase/migrations/` directory)
- Connection via `SUPABASE_DATABASE_URL` environment variable

**Authentication:**

- **Better Auth** library with Supabase PostgreSQL adapter
- JWT tokens for session management (3-day default, 30-day with "remember me")
- OAuth 2.0 integration (Google Cloud Platform, Microsoft Azure AD)
- Bcrypt password hashing (cost factor 12)
- Email verification required for registration

**Security Features:**

- Account lockout after 5 failed login attempts (30-minute lock)
- Rate limiting: 10 login attempts/min per IP, 3 password resets/hr per email
- Trial period: 14 days from email verification
- Soft delete with `is_deleted` flag

### Database Schema

**Authentication tables** (5 tables):

- `users` - Main user accounts with subscription tiers and trial tracking
- `oauth_providers` - Social login provider mappings
- `password_resets` - Password reset tokens (1-hour expiration)
- `email_verifications` - Email verification tokens (24-hour expiration)
- `auth_logs` - Security audit trail (7-day retention)

**Client Hub tables** (5 tables):

- `clients` - Client profiles with soft delete
- `client_tags` - Flexible tagging system for organization
- `client_documents` - Document storage with chronological sorting (no categories)
- `client_notes` - Activity and interaction notes
- `client_tasks` - Task management with due dates and priorities

## Project Management System (CCPM)

This repository uses Claude Code Project Management (CCPM) - a markdown-based project management system integrated into the `.claude/` directory.

### Directory Structure

```
.claude/
├── prds/                    # Product Requirements Documents
│   ├── client-hub.md
│   └── user-authentication.md
├── epics/                   # Technical implementation plans
│   └── user-authentication/
│       ├── epic.md          # Technical plan and architecture
│       ├── 001.md           # Task: Database schema
│       ├── 002.md           # Task: Better Auth integration
│       └── ...              # Additional tasks
├── commands/                # Custom slash commands
│   ├── pm/                  # Project management commands
│   ├── context/             # Context management
│   └── testing/             # Test execution
├── rules/                   # Development guidelines
│   ├── datetime.md          # ISO 8601 datetime standards
│   ├── frontmatter-operations.md
│   ├── path-standards.md
│   ├── github-operations.md
│   └── standard-patterns.md
├── skills/                  # Reusable skill definitions
├── agents/                  # Specialized agent configurations
└── ccpm.config              # GitHub repository configuration
```

### Key PM Commands

**Creating Work:**

```bash
/pm:prd-new <name>           # Create Product Requirements Document
/pm:prd-parse <name>         # Convert PRD to technical epic
/pm:epic-decompose <name>    # Break epic into tasks
```

**Viewing Status:**

```bash
/pm:status                   # Project dashboard
/pm:epic-show <name>         # View epic and all tasks
/pm:prd-list                 # List all PRDs
/pm:next                     # Show next priority tasks
```

**GitHub Sync (Optional):**

```bash
/pm:epic-sync <name>         # Push epic and tasks to GitHub Issues
/pm:sync                     # Bidirectional sync with GitHub
```

### Local-Only Mode

CCPM works entirely offline without GitHub integration. All project management is done through local markdown files with YAML frontmatter. See `LOCAL_MODE.md` for complete local workflow.

### File Frontmatter Standards

All markdown files use YAML frontmatter with ISO 8601 timestamps:

```yaml
---
name: feature-name
status:
  open # PRDs: backlog/in-progress/complete
  # Epics: backlog/in-progress/completed
  # Tasks: open/in-progress/closed
created: 2026-01-06T08:26:55Z
updated: 2026-01-06T08:58:07Z
depends_on: [001, 002] # Task dependencies
parallel: true # Can run in parallel
---
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silver0510/solidseed](https://github.com/silver0510/solidseed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
