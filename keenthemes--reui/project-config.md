---
trigger: always_on
description: - Be concise. Skip unnecessary explanations
---

# ReUI v2 - Cursor Rules

## Agent Response Rules

- Be concise. Skip unnecessary explanations
- No filler phrases ("I'll help you", "Let me", "Sure", etc.)
- Show code, not descriptions of code
- One-line answers when possible
- No repeating back what user asked
- Skip "Here's what I did" summaries unless complex
- Use bullet points over paragraphs
- Don't explain obvious changes
- Always use the most relevant available skills before implementing
- Never copy reference UI verbatim; reinterpret it into an original composition
- For product UI, prefer proven ReUI/shadcn patterns over ad-hoc custom markup
- Keep blocks responsive by default with strong mobile behavior and polished UX
- Favor clean code, extracted reuse, and minimal Tailwind classes
- In `meta.json`, keep `description` to 5-6 words max and make it the clearest summary of the block’s main feature

## Project Overview

ReUI is a React UI component library built on shadcn/ui patterns, providing accessible, customizable components styled with Tailwind CSS v4. The project powers the **shadcn create** customizer experience.

### Key Architecture

- **Multi-Base Support**: Components available in `base` (Base UI) and `radix` (Radix UI) variants
- **Multi-Style Support**: 5 visual styles - `vega`, `nova`, `maia`, `lyra`, `mira`
- **Style Name Format**: `{base}-{style}` (e.g., `radix-nova`, `base-vega`)
- **Registry System**: Serves components via API for `npx shadcn add @reui/component`

### Core Systems

1. **Customizer** (`app/(create)/`): Interactive design system configuration
2. **Pattern Registry** (`registry-reui/`): 600+ pattern components
3. **Core Registry** (`registry/`): Base shadcn components, styles, icons
4. **API Layer** (`app/api/`, `app/r/`): Registry serving for CLI integration

## Tech Stack

- **Framework**: Next.js 15+ with App Router and Turbopack
- **Language**: TypeScript (strict mode)
- **React**: React 19 with RSC (React Server Components)
- **Styling**: Tailwind CSS v4 with CSS variables
- **UI Primitives**: @base-ui/react, radix-ui
- **State Management**:
  - `nuqs` - URL query state (shareable/deep-linkable)
  - `jotai` with `atomWithStorage` - localStorage persistence
  - React Context - runtime overrides
- **Style Transformation**: `shadcn/utils` for cn-* → Tailwind conversion
- **Variants**: class-variance-authority (CVA)
- **Icons**: lucide-react, @hugeicons/react, @tabler/icons-react, phosphor-icons, remixicon
- **Forms**: react-hook-form with @hookform/resolvers, zod validation
- **Animation**: motion (Framer Motion)

## Directory Structure

```
oss/
├── app/
│   ├── (create)/                    # Customizer interface
│   │   ├── components/              # Pickers, providers, UI
│   │   │   ├── design-system-provider.tsx  # State providers
│   │   │   ├── icon-placeholder.tsx        # Multi-library icon component
│   │   │   ├── *-picker.tsx               # Config pickers (11 total)
│   │   │   └── picker.tsx                 # Base picker primitives
│   │   ├── patterns/                # Pattern browsing
│   │   │   ├── components/          # Grid, cards, sidebar, search
│   │   │   ├── [category]/          # Category pages
│   │   │   └── page.tsx             # Main patterns page
│   │   ├── preview/                 # Iframe preview pages
│   │   │   └── [base]/patterns/     # Base-specific previews
│   │   ├── hooks/                   # use-iframe-sync, use-locks
│   │   └── lib/                     # search-params, fonts, utils
│   ├── (app)/                       # Main app routes
│   │   ├── (root)/                  # Homepage
│   │   ├── docs/                    # Documentation
│   │   └── icons/                   # Icon library browser
│   ├── api/
│   │   └── registry/[name]/         # Internal registry API
│   └── r/[...path]/                 # Public CLI API (ISR)
├── registry/                        # Core shadcn components
│   ├── bases/                       # Components per base
│   │   ├── base/                    # Base UI components
│   │   │   ├── ui/                  # Primitives
│   │   │   ├── examples/            # Component examples
│   │   │   └── blocks/              # Block components
│   │   └── radix/                   # Radix UI components
│   ├── styles/                      # Style CSS files
│   │   └── style-{style}.css        # vega, nova, maia, lyra, mira
│   ├── icons/                       # Icon library wrappers
│   │   ├── icon-lucide.tsx
│   │   ├── icon-hugeicons.tsx
│   │   ├── icon-tabler.tsx
│   │   ├── icon-phosphor.tsx
│   │   └── icon-remixicon.tsx
│   ├── config.ts                    # Bases, styles, themes, fonts config
│   ├── themes.ts                    # Theme definitions with CSS vars
│   └── bases.ts                     # Base definitions
├── registry-reui/                   # ReUI pattern registry
│   ├── bases/
│   │   ├── base/                    # Base UI patterns
│   │   │   ├── patterns/            # Pattern components by category
│   │   │   │   ├── alert/           # c-alert-1.tsx, c-alert-2.tsx...
│   │   │   │   ├── button/
│   │   │   │   └── .../             # 40+ categories
│   │   │   ├── reui/                # ReUI-specific components
│   │   │   │   ├── data-grid/       # Complex data grid
│   │   │   │   ├── kanban/
│   │   │   │   └── .../

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keenthemes/reui](https://github.com/keenthemes/reui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
