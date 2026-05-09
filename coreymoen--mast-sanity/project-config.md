---
trigger: always_on
description: This document contains project-specific instructions and context for Claude Code when working on this codebase.
---

# Claude Code Project Instructions

This document contains project-specific instructions and context for Claude Code when working on this codebase.

## Project Overview

This is a **Mast design system** implementation using:
- **Frontend**: Next.js 15 with App Router
- **CMS**: Sanity Studio v3 with Visual Editing/Presentation mode
- **Styling**: Tailwind CSS v4 with CSS-first configuration

## Environment Constraints

### No esbuild/tsx on this machine
The `npx tsx` command does not work due to esbuild issues. When creating seed scripts or Node.js utilities:
- Use `.mjs` extension with ES modules (`import`/`export`)
- Run with plain `node scripts/filename.mjs`
- Avoid TypeScript for scripts that need to run directly

### Dev Servers via Docker/OrbStack
**IMPORTANT**: Dev servers cannot be started directly from the terminal on this machine. They must be run via Docker containers in the OrbStack app.

- **Do NOT** attempt to run `npm run dev`, `npm run dev:next`, or `npm run dev:studio` directly
- The user manages dev servers through OrbStack's Docker interface
- Frontend runs in Docker on port 4000, Studio on port 3333
- For verification, use TypeScript compilation (`npx tsc --noEmit`) instead of starting dev servers
- If the user needs to restart servers, they will do it manually via OrbStack

## Sanity Content Architecture

### Page Structure Hierarchy
```
Page
└── pageBuilder (array)
    └── section
        └── rows (array)
            └── row
                └── columns (array)
                    └── column
                        └── content (array of blocks)
```

### Sanity API Nesting Limit
**CRITICAL**: Sanity has a **maximum attribute depth of 20 levels**. When creating pages programmatically, you must carefully track nesting depth to avoid hitting this limit.

#### Depth Calculation Reference
Here's how nesting depth accumulates in this page builder:

```
Level 1:  page
Level 2:  └── pageBuilder (array)
Level 3:      └── section
Level 4:          └── rows (array)
Level 5:              └── row
Level 6:                  └── columns (array)
Level 7:                      └── column
Level 8:                          └── content (array)
Level 9:                              └── [block]
```

**Base path to a simple block: 9 levels**

Block types add additional depth based on their internal structure:
| Block Type | Additional Depth | Total from Page | Notes |
|------------|------------------|-----------------|-------|
| headingBlock | +1 (text field) | 10 | Safe |
| richTextBlock | +4 (content→block→children→text) | 13 | Safe |
| buttonBlock | +3 (link→href/page/post) | 12 | Safe |
| imageBlock | +3 (image→asset→_ref) | 12 | Safe |
| cardBlock | +2 (content array + block) | 11+ | ⚠️ Adds nesting |
| tabsBlock | +4 (tabs→tab→content→block) | 13+ | ⚠️ Risky |
| accordionBlock | +3 (items→item→content) | 12+ | ⚠️ Adds nesting |
| sliderBlock | +3 (slides→slide→image) | 12 | Safe |

#### Safe Nesting Patterns
```
✅ section → row → column → headingBlock (10 levels)
✅ section → row → column → richTextBlock (13 levels)
✅ section → row → column → imageBlock (12 levels)
✅ section → row → column → accordionBlock → richTextBlock (16 levels)
```

#### Unsafe Nesting Patterns
```
❌ section → row → column → tabsBlock → row → column → cardBlock → richTextBlock (19+ levels)
❌ section → row → column → cardBlock → tabsBlock → content (18+ levels)
❌ Deeply nested Portable Text with multiple marks and annotations
```

#### Best Practices
1. **Avoid nesting containers**: Don't put tabsBlock inside cardBlock or vice versa
2. **Prefer flat layouts**: Use multiple columns with icons instead of nested cards
3. **Limit accordion/tab content**: Keep content inside accordions/tabs simple (headings, text, buttons)
4. **Test before deploying**: Always run seed scripts to verify complex layouts don't exceed limits
5. **Use the validation helper**: When writing seed scripts, consider adding depth tracking

#### Depth Validation Helper (Optional)
For complex seed scripts, you can add a depth checker:
```javascript
function checkDepth(obj, maxDepth = 20, currentDepth = 0, path = '') {
  if (currentDepth > maxDepth) {
    console.warn(`⚠️ Depth ${currentDepth} exceeds ${maxDepth} at: ${path}`)
    return false
  }
  if (obj && typeof obj === 'object') {
    for (const [key, value] of Object.entries(obj)) {
      if (!checkDepth(value, maxDepth, currentDepth + 1, `${path}.${key}`)) {
        return false
      }
    }
  }
  return true
}

// Usage: checkDepth(pageDocument)
```

### Creating Pages via Script

1. Create a `.mjs` file in `/scripts/`
2. Use the `@sanity/client` package
3. Requires `SANITY_API_TOKEN` environment variable with write permissions
4. Run with: `SANITY_API_TOKEN="your-token" node scripts/your-script.mjs`

Example seed script pattern:
```javascript
import {createClient} from '@sanity/client'

const client = createClient({
  projectId: '6lj3hi0f',
  dataset: 'production',
  token: process.env.SANITY_API_TOKEN,
  apiVersion: '2024-01-01',
  useCdn: false,
})

// Helper to generate unique keys (required for all array items)
const generateKey = () => Math.random().toString(36).substring(2, 12)

// All nested objects need _type and _key
const page = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoreyMoen/mast-sanity](https://github.com/CoreyMoen/mast-sanity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
