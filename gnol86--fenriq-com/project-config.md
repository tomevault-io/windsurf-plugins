---
trigger: always_on
description: This file provides specific guidance for specialized agents working with this codebase.
---

# AGENTS.md

This file provides specific guidance for specialized agents working with this codebase.

## About the project Boilerplate

Boilerplate for Next.js 16 projects with Prisma, Better-Auth, Resend, Tailwind CSS 4, Shadcn, lucide-react, React-Hook-Form, zod, Next-Themes, and more.

## Development Commands

### Core Commands

- `bun dev` - Start development server with Turbopack
- `bun build` - Build the application
- `bun start` - Start production server
- `bun lint` - Lint the code
- `bun format` - Format the code

## Architecture Overview

### Technology Stack

- **Framework**: Next.js 15.5 with App Router
- **Language**: Javascript
- **Styling**: TailwindCSS v4 with Shadcn/UI components
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Better-Auth with organization and admin support
- **Email**: React Email with Resend
- **Payments**: Stripe integration
- **Package Manager**: bun

### Project Structure

- `app/` - Next.js App Router pages and layouts
- `src/components/` - UI components (Shadcn/UI in `src/components/ui/`, custom in `src/components/`)
- `src/features/` - Feature-specific components and logic
- `src/lib/` - Utilities, configurations, and services
- `src/hooks/` - Custom React hooks
- `src/actions/` - Actions server
- `src/components/email/` - Email templates using React Email
- `prisma/` - Database schema and migrations
- `src/messages/` - Internationalization messages

### Key Features

- **Multi-tenant Organizations**: Full organization management with roles and permissions
- **Authentication**: Email/password
- **Billing**: Stripe subscriptions with plan management
- **Dialog System**: Global dialog manager for modals and confirmations
- **Forms**: React Hook Form with Zod validation and server actions
- **Email System**: Transactional emails with React Email

## Multi-Project Architecture

This boilerplate supports creating multiple projects (A, B, C) that can receive updates from the boilerplate upstream without conflicts.

### Project vs Boilerplate Files

**🔒 Protected Project Files** (never overwritten by upstream updates):

- `src/project/**` - ALL your custom code (components, actions, hooks, lib, features, sidebar)
- `src/messages/*.project.json` - Project-specific translations (fr, en, nl, de)
- `prisma/schema/project.prisma` - Project-specific database models
- `src/app/page.js` - Landing page template
- `src/app/globals.css` - Theme/styling customization
- `src/site-config.js` - Project configuration
- `public/images/logo.png` - Your logo
- `public/images/icon.png` - Your icon

**⬆️ Boilerplate Files** (automatically updated from upstream):

- `src/actions/*.action.js` - Boilerplate server actions
- `src/components/**` - Boilerplate UI components
- `src/lib/**` - Boilerplate utilities
- `src/hooks/**` - Boilerplate hooks
- `src/app/(pages)/**` - Dashboard pages
- `src/messages/*.json` - Boilerplate translations (not \*.project.json)
- `prisma/schema/base.prisma` - Boilerplate database models

### Import Paths

Use `@project/*` for project-specific code:

```js
// Boilerplate imports
import { Button } from "@/components/ui/button";
import { useServerAction } from "@/hooks/use-server-action";

// Project imports
import { MyComponent } from "@project/components/my-component";
import { myAction } from "@project/actions/my.action";
import { useMyHook } from "@project/hooks/use-my-hook";
```

### Where to Put Your Code

**✅ ALWAYS put project-specific code in `src/project/`:**

- Components: `src/project/components/my-component.jsx`
- Server actions: `src/project/actions/my.action.js`
- Hooks: `src/project/hooks/use-my-hook.js`
- Utilities: `src/project/lib/my-util.js`
- Features: `src/project/features/my-feature/`
- Sidebar: `src/project/sidebar/app-sidebar.jsx`

**✅ Project translations in `*.project.json`:**

```json
// src/messages/en.project.json
{
    "my_feature": {
        "title": "My Feature",
        "description": "..."
    }
}
```

**Usage:**

```js
const t = useTranslations("project.my_feature");
t("title"); // "My Feature"
```

**✅ Project database models in `prisma/schema/project.prisma`:**

```prisma
model Product {
    id             String       @id @default(cuid())
    name           String
    organizationId String
    organization   Organization @relation(fields: [organizationId], references: [id])

    @@map("product")
}
```

### Updating from Boilerplate

To get the latest boilerplate updates:

```bash
# Fetch and merge upstream changes
git fetch upstream
git merge upstream/main

# Install new dependencies if any
bun install

# Regenerate Prisma if schema changed
bun prisma generate
```

**What happens:**

- ✅ Boilerplate files automatically updated
- ✅ Project files (`src/project/**`) never touched
- ⚠️ `package.json` may need manual merge (keep both dependencies)

### Setup New Project

See `.github/SETUP_NEW_PROJECT.md` for the complete guide.

**Quick start:**

```bash
./scripts/setup-new-project.sh
```

**What the script does:**

1. Clones the boilerplate into a new directory
2. Configures `upstream` (boilerplate) and `origin` (your project) remotes
3. Activates `merge=theirs` strategy (defined in `.gitattributes`) to protect project files
4. Updates `src/site-config.js` with the project name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gnol86/fenriq.com](https://github.com/Gnol86/fenriq.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
