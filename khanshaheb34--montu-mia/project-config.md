---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "মন্টু মিয়াঁর সিস্টেম ডিজাইন" (Montu Mia's System Design) - a Bengali-language system design learning platform. The project teaches system design concepts through storytelling, using simple analogies and Bengali language content.

## Commands

### Development

```bash
bun install          # Install dependencies
bun dev              # Start development server at http://localhost:3000
bun build            # Build for production
bun start            # Start production server
```

### Code Quality

```bash
bun run types:check  # Run full type checking (includes MDX processing and Next.js type generation)
bun run lint         # Run Biome linter
bun run format       # Format code with Biome
```

### Email & Newsletter

```bash
bun run email:preview      # Preview email template in browser at http://localhost:3001
bun run send-emails:test   # Send test email to shakirulhkhan@gmail.com
bun run send-emails:all    # Send to all subscribers (requires confirmation)
```

### OG Image Generation

```bash
bun run generate-og    # Generate OG images for all MDX content
```

### Post-Install

- `fumadocs-mdx` runs automatically after `bun install` via postinstall script

## Architecture

### Content Management (Fumadocs)

The project uses **Fumadocs** - a documentation framework built on Next.js. Content workflow:

1. **Content Source**: MDX files in `content/sd/` directory
   - Example: `content/sd/introduction.mdx`, `content/sd/scaling.mdx`
   - Navigation structure defined in `content/sd/meta.json`

2. **Content Processing Pipeline**:
   - `source.config.ts` defines the docs collection pointing to `content/sd/`
   - `fumadocs-mdx` (run via postinstall) processes MDX files and generates output to `.source/` directory
   - The `.source/` directory is gitignored and regenerated on each build

3. **Content Access**:
   - Import from `"fumadocs-mdx:collections/server"` (virtual import)
   - This path maps to `.source/*` via tsconfig path alias
   - Used in `src/lib/source.ts` which exports the `source` loader

4. **Rendering**:
   - `src/app/sd/[[...slug]]/page.tsx` - Catch-all route for rendering docs pages
   - `src/app/sd/layout.tsx` - Uses Fumadocs `DocsLayout` component
   - MDX components defined in `src/mdx-components.tsx`

### App Structure (Next.js App Router)

- `src/app/(home)/` - Homepage route group
- `src/app/sd/` - System design docs section (main content)
- `src/app/api/search/` - Search API endpoint
- `src/app/og/` - Dynamic OG image generation
- `src/app/actions.ts` - Server actions (newsletter subscription via AWS SES)

### Key Libraries

- **Fumadocs**: Documentation framework (fumadocs-core, fumadocs-mdx, fumadocs-ui)
- **Next.js 16**: App Router with React Server Components
- **Tailwind CSS 4**: Styling via PostCSS
- **Biome**: Linting and formatting
- **AWS SES**: Email service for newsletter subscriptions (eu-west-1 region)
- **Radix UI**: Headless UI components (Dialog, Slot, etc.)
- **Vercel Analytics & Speed Insights**: Built-in tracking

### Styling

- Uses two fonts: Outfit (Latin) and Noto Sans Bengali (Bengali script)
- Font variables: `--font-outfit`, `--font-bengali`
- Tailwind configured via `@tailwindcss/postcss` plugin
- Custom component utilities in `src/lib/cn.ts` (class merging)

### Newsletter Feature

The newsletter subscription flow:

1. User clicks subscribe button (rendered via `SubscribeModal` component)
2. Modal opens with email input form
3. Form submission triggers `subscribeToNewsletter` server action in `src/app/actions.ts`
4. Server action uses AWS SES `CreateContactCommand` to add contact to SES contact list
5. Requires environment variables: `AWS_REGION`, `SES_CONTACT_LIST_NAME`

### Email System (Newsletter Sending)

The project includes a complete email sending pipeline using React Email and AWS SES:

1. **Email Templates**:
   - Located in `emails/` directory
   - `newsletter-react.tsx` - Main React Email template
   - `data/past-posts.json` - List of past articles to include in emails
   - Built with `@react-email/components` for email client compatibility

2. **Template Features**:
   - Site logo/OG image at top
   - Bengali title: "মন্টু মিয়াঁর নতুন অভিযানে স্বাগতম"
   - Last episode summary (variable)
   - Current topic teaser (variable)
   - Featured article image (variable)
   - LinkedIn article link (variable)
   - Auto-generated past posts list from JSON
   - Unsubscribe link
   - Matches site theme from `global.css` (amber/yellow color scheme)

3. **Sending Emails**:

   ```bash
   # Preview email in browser (with hot reload)
   bun run email:preview    # Opens http://localhost:3001

   # Send test email to shakirulhkhan@gmail.com
   bun run send-emails:test

   # Send to all subscribers in SES contact list
   bun run send-emails:all

   # List current subscribers
   bun run subscribers [count]
   ```

4. **Customization**:
   - Edit `scripts/send-emails.ts` to update:
     - `EMAIL_CONTENT` object (subject, summary, links, images)
     - `TEST_RECIPIENT` or add multiple recipients
     - Rate limiting delay (default: 1 second between sends)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KhanShaheb34/montu-mia](https://github.com/KhanShaheb34/montu-mia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
