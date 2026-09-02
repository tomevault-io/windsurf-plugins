---
trigger: always_on
description: Never use eslint disables for TSC errors. Fix the problem. If you don't know the type, use unknown, never use any.
---

# PRIME DIRECTIVES
Never use eslint disables for TSC errors. Fix the problem. If you don't know the type, use unknown, never use any.

# CURSOR DIRECTIVES
## Component Creation Priority
1. **ALWAYS check RFDS first**: `import { RFDS } from "@/components/rfds"`
2. **Use Semantic Components**: `RFDS.SemanticButton`, `RFDS.SemanticCard`, `RFDS.SemanticBadge`
3. **Compose from Primitives**: If custom needed, use `RFDS.Button`, `RFDS.Pill`, `RFDS.Rating`
4. **One-off components**: Still use design system primitives as building blocks
5. **NEVER create from scratch**: Always start with design system components

## Color Usage Rules
- **NEVER use hardcoded colors**: `bg-blue-500`, `text-red-600`, `border-gray-200`
- **ALWAYS use semantic colors**: `bg-primary`, `text-destructive`, `border-border`
- **Check theme-config.ts**: For available semantic color tokens
- **Test both themes**: Light and dark mode compatibility required

## Default Import Pattern
```typescript
// ✅ CORRECT - Start with design system
import { RFDS } from "@/components/rfds";

// Use semantic components first
<RFDS.SemanticButton variant="primary">Click me</RFDS.SemanticButton>
<RFDS.SemanticCard>Content</RFDS.SemanticCard>

// Compose from primitives if needed
<div className="bg-card text-card-foreground border border-border">
  <RFDS.Button variant="primary">Action</RFDS.Button>
</div>
```

# Claude Directives for React Foundation Store

## 🎯 CRITICAL RULES (NEVER BREAK)

1. **TypeScript MUST pass**: Run `npx tsc --noEmit` before claiming ANY task is complete
2. **Tests MUST pass**: Run `npm run lint` - NO EXCEPTIONS before commit
3. **Fix ALL warnings**: Warnings exist for a reason - fix them, don't ignore them
4. **NO console.log**: Use proper logging or remove debug statements
5. **Work incrementally**: Debug ONE thing at a time, prove it works, then move forward
6. **Document progress**: Create `/todos/<branch-name>-todo.md` for multi-session work

## 🏗️ ARCHITECTURE

### Project Structure
```
react-foundation-store/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── page.tsx           # Foundation homepage
│   │   ├── about/             # About the Foundation
│   │   ├── impact/            # Impact reports
│   │   ├── store/             # 🛒 Store section
│   │   │   ├── page.tsx       # Store home
│   │   │   ├── products/      # Product detail pages
│   │   │   └── collections/   # Collection pages
│   │   └── api/               # API routes (shared)
│   ├── components/
│   │   ├── rfds/              # React Foundation Design System
│   │   ├── layout/            # Header, Footer
│   │   ├── ui/                # UI primitives
│   │   └── home/              # Home page components
│   ├── lib/                   # Utilities & data
│   │   ├── ris/               # React Impact Score system
│   │   ├── shopify.ts         # Shopify integration
│   │   └── providers/         # Context providers
│   └── types/                 # TypeScript definitions
├── scripts/                   # Shopify management scripts
├── docs/                      # Documentation
│   ├── foundation/            # Foundation docs
│   └── store/                 # Store docs
└── public/                    # Static assets
```

### Tech Stack
- **Framework**: Next.js 15 (App Router, React 19, Turbopack)
- **CMS**: Shopify (Storefront + Admin APIs)
- **Authentication**: NextAuth with GitHub OAuth
- **3D Graphics**: Three.js, React Three Fiber, React Three Drei
- **AI**: OpenAI GPT-5 Vision, gpt-image-1
- **Styling**: Tailwind CSS 4
- **Language**: TypeScript 5

## 📝 CODE QUALITY

### TypeScript Standards
- **Complete interfaces**: Define ALL required properties
- **Proper unions**: Use `'A' | 'B' | 'C'` not `'A' as const`
- **No `any`**: All types defined in `types/` folder
- **Match definitions**: Return types must match interface exactly
- **Strict mode**: Always use strict TypeScript configuration

### Next.js App Router Patterns
- **Server Components**: Default to server components, use `'use client'` only when needed
- **Route handlers**: API routes in `app/api/` directory
- **Layouts**: Use layout.tsx for shared UI
- **Loading/Error**: Implement loading.tsx and error.tsx
- **Metadata**: Use generateMetadata for SEO

### React Foundation Design System (RFDS)
```typescript
import { RFDS } from "@/components/rfds"

// Use RFDS components consistently
<RFDS.Button variant="primary" size="lg">Click me</RFDS.Button>
<RFDS.ProductCard product={product} />
<RFDS.Header />
```

**Architecture Layers:**
- **Primitives**: Base building blocks (Button, Typography, etc.)
- **Components**: Composed from primitives (ProductCard, etc.)
- **Layouts**: Page structure (Header, Footer)
- **Semantic Components**: Themeable components (SemanticButton, SemanticCard, etc.)

## 🎨 DESIGN SYSTEM & THEMING

### CRITICAL: Always Use Design System
**NEVER create new components without checking the design system first!**

#### Design System Hierarchy
1. **Check RFDS first**: `import { RFDS } from "@/components/rfds"`
2. **Use Semantic Components**: For themeable, consistent UI
3. **Compose from Primitives**: If you need something custom
4. **One-off components**: Still use design system primitives


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thereactfoundation/react.foundation](https://github.com/thereactfoundation/react.foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
