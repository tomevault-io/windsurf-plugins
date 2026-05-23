---
trigger: always_on
description: DevVerse SWE Blog is a modern tech blog built with Next.js, React, TypeScript, MDX, Supabase, and TailwindCSS. It serves as a platform for sharing software engineering insights and technical tutorials.
---

# DevVerse SWE Blog - GitHub Copilot Instructions

DevVerse SWE Blog is a modern tech blog built with Next.js, React, TypeScript, MDX, Supabase, and TailwindCSS. It serves as a platform for sharing software engineering insights and technical tutorials.

**CRITICAL: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Bootstrap and Setup
```bash
# Install dependencies
npm install  # Takes ~30-40 seconds, NEVER CANCEL
```

**ENVIRONMENT VARIABLES REQUIRED:**
```bash
# Create .env file or set environment variables for development
NEXT_PUBLIC_SUPABASE_URL=https://example.supabase.co  # Use stub for development
NEXT_PUBLIC_SUPABASE_ANON_KEY=example_key             # Use stub for development
```

**CRITICAL: Without these environment variables, the application will crash with "supabaseUrl is required" error.**

### Development Server
```bash
# Start development server with environment variables
NEXT_PUBLIC_SUPABASE_URL=https://example.supabase.co NEXT_PUBLIC_SUPABASE_ANON_KEY=example_key npm run dev
# OR set environment variables first, then run:
npm run dev  # Server starts on http://localhost:3000
```
- Takes ~1-2 seconds to start
- **ALWAYS** set environment variables before running dev server
- App crashes without proper Supabase environment variables

### Testing and Validation
```bash
# Run tests - NEVER CANCEL
npm run test  # Takes 1-2 seconds, all 11 tests should pass

# Run tests with coverage
npm run coverage  # Takes 1-2 seconds, generates coverage reports

# Run tests in watch mode
npm run test:watch
```

### Linting and Formatting
```bash
# Format code (this is also the lint command)
npm run lint      # Runs prettier, takes ~1-2 seconds
npm run format    # Same as lint, runs prettier --write
```
**ALWAYS run `npm run format` before committing changes - the CI pipeline will fail without proper formatting.**

### Build Process
**⚠️ CRITICAL BUILD LIMITATION:**
```bash
npm run build  # FAILS in sandboxed environments due to Google Fonts access
```
**NEVER CANCEL the build command - it will timeout due to network restrictions trying to fetch fonts from fonts.googleapis.com. This is expected in sandboxed environments.**

The build failure is due to:
- `next/font/google` imports in `app/layout.tsx` and `components/TableOfContents.tsx`
- Network restrictions preventing access to Google Fonts API
- This does NOT affect development server functionality

## Manual Validation Requirements

### Essential Validation Steps
After making changes, **ALWAYS** run through these validation scenarios:

1. **Start Development Server:**
   ```bash
   NEXT_PUBLIC_SUPABASE_URL=https://example.supabase.co NEXT_PUBLIC_SUPABASE_ANON_KEY=example_key npm run dev
   ```

2. **Navigate to http://localhost:3000 and verify:**
   - Homepage loads with article list
   - Search functionality works
   - Topic filtering works
   - At least one article can be clicked and loads properly

3. **Test Article Navigation:**
   - Click on any article from the homepage
   - Verify article content loads with proper formatting
   - Verify code blocks display correctly
   - Verify table of contents appears and works (desktop)
   - Verify breadcrumb navigation works

4. **Run Tests:**
   ```bash
   npm run test && npm run coverage
   ```

5. **Format Code:**
   ```bash
   npm run format
   ```

## Project Structure and Key Areas

### Core Application Structure
```
├── app/                    # Next.js 13+ app directory
│   ├── api/               # API routes (reset-password, verify-email, feeds)
│   ├── articles/[slug]/   # Dynamic article pages
│   ├── auth/             # Authentication pages
│   ├── layout.tsx        # Root layout (contains Google Fonts imports)
│   └── page.tsx          # Homepage
├── content/              # MDX blog articles (40+ articles)
├── components/           # React components
│   ├── Navbar.tsx       # Main navigation
│   ├── TableOfContents.tsx  # Article TOC (contains Google Fonts)
│   └── [others]         # Various UI components
├── lib/                 # Utility libraries
│   ├── articles.ts      # Article processing logic
│   ├── rss.ts          # RSS feed generation
│   └── jsonfeed.ts     # JSON feed generation
├── tests/              # Jest test files
├── .github/            # GitHub workflows and templates
│   └── workflows/ci.yml # CI/CD pipeline
└── [config files]     # Various configuration files
```

### Important Files to Monitor
- **`supabase/supabaseClient.ts`**: Supabase client configuration - requires environment variables
- **`app/layout.tsx`**: Root layout with Google Fonts imports - causes build issues
- **`components/TableOfContents.tsx`**: Contains Google Fonts imports - causes build issues
- **`package.json`**: Contains all available scripts and dependencies
- **`.github/workflows/ci.yml`**: CI pipeline with build, test, and deployment steps

## Common Tasks and Commands

### Available npm Scripts
```bash
npm run dev        # Start development server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoangsonww/DevVerse-SWE-Blog](https://github.com/hoangsonww/DevVerse-SWE-Blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
