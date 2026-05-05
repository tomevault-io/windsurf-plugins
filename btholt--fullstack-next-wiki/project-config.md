---
trigger: always_on
description: This is a **minimal, educational wiki app** built for Frontend Masters students. The app should be barebones, readable, and focus on core concepts rather than best practices or optimizations.
---

# Frontend Masters Fullstack Next.js Wiki - Barebones Starter

This is a **minimal, educational wiki app** built for Frontend Masters students. The app should be barebones, readable, and focus on core concepts rather than best practices or optimizations.

## Project Purpose

This is a **starter app** that Copilot builds, then **students will code along** during the course to transform it into a production-ready application. The instructor will guide students through upgrading each stub into real implementations.

**Important**: This code is designed to be **modified by humans** after Copilot finishes. The instructor and students will work together to replace stubs with real services during live coding sessions.

## Project Goals

- **Educational first**: Code should be simple and readable for students following along
- **Barebones MVP**: Bare minimum functionality, no bells and whistles
- **Easy to upgrade**: Each stub should be easily replaceable with real implementations during class
- **Working from day 1**: Students get a functional app to start with, then improve it together
- **Clear upgrade paths**: Students should see exactly what they'll be replacing and why

## Code Style Guidelines

- **Readable over optimal**: Choose simple, clear code over performant/best practice code
- **Minimal abstractions**: Avoid complex patterns, keep things straightforward
- **No premature optimization**: No caching, lazy loading, or performance tricks
- **Explicit over magic**: Prefer obvious code over clever shortcuts
- **TypeScript basics**: Use TypeScript but keep types simple and obvious

## Tech Stack (Barebones Version)

- **Framework**: Next.js 15 with App Router
- **Styling**: Tailwind CSS (pre-configured, minimal custom CSS)
- **UI Components**: One page uses plain HTML/Tailwind, one page uses shadcn/ui components
- **Database**: In-memory JavaScript objects (not real database)
- **Auth**: Stub functions that return hardcoded users/roles
- **File Upload**: Saves to local /uploads folder, no processing
- **Email**: Stub functions that console.log instead of sending
- **AI**: Stub function that returns the input string
- **Tests**: Vitest and Playwright configured but empty
- **Deployment**: No deployment config (students will add this)

## File Structure Conventions

### Steps

The way this course is structured, each folder (e.g. 00-start, 01-shadcn, etc. to 09-with-tests) is one step in the project. Inside of each step folder is the complete state of the project. This corresponds to checkpoints in the course.

The full course's text can be found at: https://fullstack-v4.holt.courses/llms.txt

Each step should incrementally build to the next. Take care that if you modify something in one step that it may need to be changed in subsequent and/or previous steps. Generally if you find something like, please flag to the user that more than one step needs to changed and ask for guidance from them.

## Common Commands

- `npm run dev`: Start development server
- `npm run build`: Build for production
- `npm run start`: Serve the finished build

## App Features (Minimal)

The wiki should have these basic features working:

- **Pages**: List all pages, view a page, create/edit pages
- **Users**: Hardcoded users with different roles (admin, editor, viewer)
- **Auth**: Fake login that switches between hardcoded users
- **Content**: Pages store title + markdown content
- **Files**: Basic file upload (save to disk, show in page)
- **Styling**: Most pages styled with Tailwind, one page uses shadcn components

## Stubs to Replace Later

These are intentionally minimal with obvious upgrade paths:

### Auth (Currently: Hardcoded → Will become: Stack Auth)

- **Current**: `/lib/auth.js` - returns hardcoded user objects, fake session management
- **Future**: Stack Auth React components, real sessions, middleware protection
- **Stub design**: Make auth functions async, return user objects with expected shape

### Database (Currently: In-memory → Will become: Neon + Drizzle)

- **Current**: `/lib/models/*.js` - manipulates JavaScript arrays in memory
- **Future**: Neon Postgres with Drizzle ORM, real schemas, migrations
- **Stub design**: All DB operations return Promises, use async/await everywhere

### Email (Currently: Console logs → Will become: Resend)

- **Current**: `/lib/email.js` - console.log instead of sending emails
- **Future**: Resend API for transactional emails (welcome, notifications)
- **Stub design**: Async functions that take email data objects, return success/failure

### AI (Currently: Echo → Will become: Vercel AI SDK)

- **Current**: `/lib/ai.js` - returns input string unchanged
- **Future**: OpenAI/Anthropic via Vercel AI SDK for content summarization
- **Stub design**: Async function that takes content string, returns "summary" string

### File Storage (Currently: Local disk → Will become: Vercel Blob)

- **Current**: Saves files to `/uploads` folder with basic file handling
- **Future**: Vercel Blob storage with URLs, metadata, proper file management
- **Stub design**: Upload functions return file objects with url, size, type properties


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btholt/fullstack-next-wiki](https://github.com/btholt/fullstack-next-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
