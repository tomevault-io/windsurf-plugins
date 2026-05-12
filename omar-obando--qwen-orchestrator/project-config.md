---
trigger: always_on
description: **BEFORE modifying ANY file, you MUST read it completely from start to finish.**
---

# AGENTS.md

## ⚠️ MANDATORY RULE #1: READ BEFORE WRITE

**BEFORE modifying ANY file, you MUST read it completely from start to finish.**

```
❌ BANNED:
- Editing a file you haven't read in full
- Assuming file contents without reading
- Modifying code based on memory or assumptions
- Skipping the read step to "save time"

✅ REQUIRED:
- Read the ENTIRE file before making ANY change
- After editing, re-read the file to verify changes are correct
- If a file is too long, read it in chunks — but read ALL of it
- Never use Edit or WriteFile on a file you haven't ReadFile'd first
```

**Why?** This prevents breaking existing functionality, duplicating code, introducing inconsistencies, and losing important logic that was already in the file.

---

## ⚠️ MANDATORY RULE #2: MULTI-PAGE WEBSITES

**When the user asks for "a website", "a site", or "a page for X business" — create a FULL multi-page website, NEVER a single landing page.**

```
❌ BANNED (for "website" requests):
- Creating index.html as the ONLY file
- Putting all content in one scrollable page
- Using anchor links (#about, #services) instead of separate routes
- No navigation between pages

✅ REQUIRED (minimum pages):
- Home (hero, value proposition, key sections)
- About (company story, team, mission)
- Services (detailed offerings with CTAs)
- Products/Portfolio (showcase work, case studies)
- Contact (form, map, phone, email, hours)
```

**The ONLY exception**: User EXPLICITLY says "landing page" or "one-page site".

Use the `design-system` skill for full page architecture, color palettes, and typography rules.

---

## ⚠️ MANDATORY RULE #3: ZERO EMOJIS + PROPER SPACING

**Professional websites NEVER use emojis. Sections NEVER touch each other.**

```
❌ BANNED:
- 🚀 🎯 💡 ✨ 🏆 emojis anywhere in website output (headings, buttons, nav, meta, content)
- Using emojis as section icons or bullet points
- Sections with less than 80px padding between them
- Footer directly touching the section above (no gap)
- More than 7 items in main navigation (saturated menu)

✅ REQUIRED:
- SVG icons from Lucide, Heroicons, or Phosphor (pick ONE, use consistently)
- Section spacing: minimum 80px top+bottom padding (use clamp for responsive)
- Footer spacing: minimum 128px top padding
- Navigation: max 7 top-level items, dropdowns for groups, secondary in footer
- Alternating section backgrounds (--color-bg / --color-surface)
```

### Service and Product Detail Pages

```
❌ BANNED:
- /services page with only a list (no individual detail pages)
- /products page with only cards (no individual detail pages)
- Generic 2-3 line descriptions per service or product

✅ REQUIRED:
- /services listing page → links to /services/web-design, /services/seo, etc.
- Each service detail: description, process, deliverables, pricing, FAQ, CTA
- /products listing page → links to /products/[slug] detail pages
- Each product detail: gallery, specs, pricing, reviews, related products
```

---

## ⚠️ MANDATORY RULE #4: TODO TRACKING + QUALITY GATES

**Every task MUST be tracked via TodoWrite. Every deliverable MUST pass quality checks.**

```
❌ BANNED:
- Completing a task without updating TodoWrite to status: "completed"
- Delivering code without running lint/syntax/type checks
- Skipping quality gates to "save time"
- Setting status: "completed" without verification evidence

✅ REQUIRED:
- Read current TodoWrite state before starting work
- Set task to status: "in_progress" when starting (include ALL tasks in the call)
- Run framework-specific quality checks BEFORE setting status: "completed"
- Set status: "completed" ONLY after all checks pass
- If checks fail, FIX issues then re-run checks before updating TodoWrite
- ALWAYS send the FULL todos array to TodoWrite (it replaces, not merges)
```

**TodoWrite API**:

```
TodoWrite({
  todos: [
    { id: "t1", content: "Task description", status: "pending" },
    { id: "t2", content: "Another task", status: "in_progress" },
    { id: "t3", content: "Completed task", status: "completed" }
  ]
})
```

**Status values**: `"pending"` (not started), `"in_progress"` (working), `"completed"` (done)

**CRITICAL**: Every TodoWrite call must include ALL tasks (not just changed ones). The tool REPLACES the entire array — it does not merge.

### Pre-Delivery Quality Gate (MANDATORY)

Before ANY task can be set to `status: "completed"`, the Code Quality Guard or the implementing agent MUST run the appropriate checks:

| Framework        | Syntax Check           | Lint                       | Type Check                     | Build/Test          |
| ---------------- | ---------------------- | -------------------------- | ------------------------------ | ------------------- |
| **Astro**        | `astro check`          | `eslint .`                 | `tsc --noEmit`                 | `astro build`       |
| **Next.js**      | `next lint`            | `eslint .`                 | `tsc --noEmit`                 | `next build`        |
| **TypeScript**   | `tsc --noEmit`         | `eslint .`                 | `tsc --strict`                 | `npm run build`     |
| **Flutter**      | `dart analyze`         | `dart analyze`             | `dart analyze`                 | `flutter build web` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Omar-Obando/qwen-orchestrator](https://github.com/Omar-Obando/qwen-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
