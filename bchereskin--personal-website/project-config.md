---
trigger: always_on
description: Personal portfolio, blog, and content platform for Brett Chereskin — COO at dub, West Point grad, Army veteran. Deployed on Vercel, backed by Supabase.
---

# Claude Code Context — Brett Chereskin's Personal Website

## Project Overview
Personal portfolio, blog, and content platform for Brett Chereskin — COO at dub, West Point grad, Army veteran. Deployed on Vercel, backed by Supabase.

## Tech Stack
- **Next.js 16** with App Router
- **React 19** + **TypeScript 5** (strict mode)
- **Tailwind CSS v4** via `@tailwindcss/postcss`
- **Supabase** for database (blog posts, comments, contacts, subscribers, shared pages)
- **Resend** for transactional email (contact form, subscriber notifications)
- **Vercel** for hosting and deployment
- Light "Field Notes" theme only (warm paper `#efe9db`, ink `#1a1814`, single ink-blue accent `#1f4a7a`, Source Serif 4 display)

## Key Commands
```bash
npm run dev      # Start dev server at localhost:3000
npm run build    # Production build
npm run lint     # Run ESLint
```

## Project Structure
```
app/
  page.tsx             # Home page
  about/               # About page
  favorites/           # NYC favorites (restaurants, Broadway, etc.)
  blog/
    page.tsx           # Blog listing (fetches from Supabase)
    posts.ts           # Supabase query functions for blog posts
    [slug]/page.tsx    # Individual post renderer (server component)
    [slug]/page.tsx    # Individual post renderer
  contact/             # Contact form page
  admin/               # Admin dashboard (protected by ADMIN_EMAIL allowlist)
    login/             # Admin login page
  shared/[slug]/       # Hosted shared pages (database-backed)
  api/
    admin/             # Admin CRUD routes (blog-posts, comments, contacts, shared-pages, notify-subscribers)
    comments/          # Public comment submission
    contact/           # Contact form handler
    subscribe/         # Newsletter subscription
    unsubscribe/       # Newsletter unsubscribe
    og/shared/[slug]/  # Dynamic OG image generation for shared pages
    publish/           # Shared page publishing endpoint
  components/          # Shared components (Navigation, Footer, MotionSection, BlogRenderer, CommentsSection, etc.)
  lib/                 # Utilities (supabase clients, sanitize, rate-limit, types)
  hooks/               # Custom React hooks (scroll animations)
  globals.css          # CSS variables, animations, utility classes
public/                # Static assets (Headshot.jpeg, logos, SVGs)
middleware.ts          # Auth middleware for admin routes
```

## Blog System
- Blog posts are stored in **Supabase** (`blog_posts` table), NOT in local files
- `app/blog/posts.ts` contains query functions (`getPublishedPosts`, `getPostBySlug`, etc.) that fetch from Supabase
- `BlogPost` interface: `id`, `slug`, `title`, `excerpt`, `date`, `readTime`, `category`, `content`, `is_published`, `visit_count`
- Content uses a custom markdown-like format parsed by `BlogRenderer.tsx`:
  - `## Heading` / `### Heading` for headers
  - `**bold**` for bold text
  - `- item` for bullet points
  - `[MODEL]` blocks for special model comparison cards
- To add a new post: use the admin dashboard (`/admin`) or insert directly into Supabase
- Posts support comments (threaded, stored in Supabase `comments` table)
- Visit tracking via `increment_blog_post_visits` RPC

## Supabase Database (project: opnsoprahgrfwjiwyvnn)
- `blog_posts` — blog content (title, slug, content, category, published status, visit count)
- `comments` — threaded reader comments on blog posts
- `contacts` — contact form submissions
- `subscribers` — newsletter email signups
- `shared_pages` — hosted HTML pages (admin-managed content platform)
- All tables have RLS enabled

## Styling Conventions
- CSS custom properties for all colors (`--primary`, `--accent`, `--neutral-*`, etc.)
- Use Tailwind arbitrary values like `bg-[var(--background)]` to reference CSS vars
- CSS vars remap legacy `--neutral-*`/`--primary` aliases onto the Field Notes palette (see top of `globals.css`)
- Prefer server components; use `'use client'` only where interactivity requires it (forms, dashboard, comments)

## Image Handling
- Local images in `/public` — use Next.js `<Image>` component
- Remote images from Unsplash are allowed via `next.config.ts`

## Security
- All user input is sanitized via `app/lib/sanitize.ts` (`escapeHtml`, `sanitizeInput`)
- HTML entities escaped in all email templates (Resend)
- CSP + security headers configured in `next.config.ts`
- Admin routes protected by `ADMIN_EMAIL` allowlist in middleware + API routes
- Supabase RLS enabled on all tables
- Rate limiting on public API routes (`app/lib/rate-limit.ts`)

## SEO
- JSON-LD structured data: `Person` + `WebSite` in root layout, `Article` on each blog post
- RSS feed at `/feed.xml`, sitemap at `/sitemap.xml`, robots at `/robots.txt`
- Per-post OpenGraph + Twitter metadata via `generateMetadata`
- Dynamic OG images for shared pages

## Preferences
- Keep code simple and avoid over-engineering
- Don't add unnecessary comments or docstrings
- Light theme only — the dark theme was fully removed in the April 2026 redesign

## Roadmap

### Quick Wins (1-2 days each)
- [ ] **/now page** — What Brett is focused on right now, updated monthly. Gives repeat visitors a reason to return.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bchereskin/personal-website](https://github.com/bchereskin/personal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
