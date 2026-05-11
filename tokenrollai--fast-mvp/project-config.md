---
trigger: always_on
description: > This template is designed for **Vibe Coding** - rapid AI-assisted development from a production-ready starting point.
---

# Repository Guidelines for AI Agents

> This template is designed for **Vibe Coding** - rapid AI-assisted development from a production-ready starting point.

## 🎯 Agent Mindset: Ship Fast, Iterate Faster

When a user asks you to build something:

1. **Don't start from scratch** - This repo has working examples for AI chat, tRPC APIs, and UI components
2. **Copy > Create** - Find the closest existing pattern and adapt it
3. **Show, don't explain** - Build working code with UI, not just API endpoints
4. **Test as you go** - Use `pnpm dev` and verify in browser after each step
5. **🗑️ Clean slate after learning** - Once you understand the structure, DELETE example files (`trpc/`, `magic/`, `openai/`)
6. **🎨 Homepage is mandatory** - EVERY demo needs a beautiful landing page (`src/app/(pages)/page.tsx`)

### Example Decision Tree

```
User: "Build me a chatbot"
├─ ✅ DO: Copy src/app/(pages)/openai/page.tsx + src/server/routers/chat.ts
├─ ✅ DO: Reuse OpenAIChatDemo.tsx component
├─ ❌ DON'T: Build a new chat UI from scratch
└─ ❌ DON'T: Create a REST API (use tRPC instead)

User: "Add a database table for users"
├─ ⚠️ PAUSE: Ask if they really need persistence (in-memory might be fine for MVP)
├─ ✅ IF YES: Create schema in src/db/schema/, run pnpm db:push
└─ ❌ DON'T: Auto-create migrations without user confirmation

User: "Make the UI look better"
├─ ✅ DO: Add shadcn/ui components (pnpm dlx shadcn@latest add [component])
├─ ✅ DO: Reference src/components/magicui/ for animations
└─ ❌ DON'T: Write custom CSS (use Tailwind utilities)
```

## Project Structure & Module Organization

- **Project Structure**: `src/app` handles routing and pages (distinguishing between `(pages)` client views and `(server)` Route Handlers), `src/components` stores reusable UI components (prioritizing subdirectories like `ui/` and `magicui/`), `src/lib` contains utilities and AI Providers, while `src/server` and `src/db` manage tRPC and Drizzle logic. Before making changes, identify the relevant directories to maintain consistency between files and responsibilities.

- **New Feature Workflow - The Fast MVP Way**: 0. **🗑️ Clean Up (2 min)**: Remove example demos after understanding structure
  - Delete `src/app/(pages)/trpc/`, `magic/`, `openai/` if building new demo
  - Keep `src/components/` (reusable UI) and `src/server/routers/` (as reference)
  - This gives you a clean slate to build YOUR demo
  1. **Discovery (2 min)**: Scan existing code for similar features
     - Check `src/app/(pages)/` for page examples
     - Check `src/server/routers/` for API patterns
     - Check `src/components/` for reusable UI
  2. **Copy & Adapt (10 min)**: Clone the closest match
     - Copy entire page structure including error handling
     - Copy tRPC router with all its error cases
     - Copy Zod schema and extend it
  3. **Integrate (5 min)**: Wire up the new code
     - Add router to `src/server/routers/_app.ts`
     - Create page in `src/app/(pages)/feature-name/page.tsx`
     - Import and use in UI
  4. **Test & Iterate (5 min)**: Verify in browser
     - Check `pnpm dev` output for errors
     - Test happy path in browser
     - Test error cases
  5. **🎨 Design Homepage (10 min)**: Make it beautiful ⚠️ REQUIRED
     - Edit `src/app/(pages)/page.tsx` with hero section
     - Add feature showcase with icons/cards
     - Include demo preview or interactive element
     - Add clear navigation to feature pages
  6. **Polish (5 min)**: Format and finalize
     - Run `pnpm format`
     - Add loading states if missing
     - Improve error messages

  **Total: ~40 minutes per demo** (vs hours from scratch)

- **Development Process**: Always iterate using `pnpm dev` or other watch scripts. Do not run `pnpm build` within a session. Prefer project-specific scripts when executing commands.

- **Dependency Management**: Use only `pnpm add/update/remove`. Keep `pnpm-lock.yaml` synchronized with `package.json`. Restart the development server after dependency changes.

- **Code Style & UI Component Strategy**:
  - **TypeScript only** - No JavaScript files
  - **Tailwind utilities** - No custom CSS files unless absolutely necessary
  - **Co-locate styles** - Keep styles with components

  **UI Component Decision Matrix**:

  ```
  Need a button/input/card?
  ├─ ✅ FIRST: Check if it's in src/components/ui/ (shadcn/ui)
  ├─ ✅ IF NOT: Run `pnpm dlx shadcn@latest add button`
  └─ ❌ LAST RESORT: Create custom component

  Need animations/effects?
  ├─ ✅ FIRST: Check src/components/magicui/ (AnimatedButton, ParticleDemo, etc.)
  ├─ ✅ IF NOT: Check magicui.design for more components
  └─ ❌ LAST RESORT: Use framer-motion directly

  Need complex layout?
  ├─ ✅ Use Tailwind flex/grid utilities
  └─ ❌ Don't create layout components
  ```

  **Speed tip**: Browse existing pages (`src/app/(pages)/*/page.tsx`) to see real usage examples

- **AI Provider - Multi-Model Support Built-in**:
  - ✅ **ALWAYS use**: `resolveLanguageModel(provider, model?)` from `src/lib/ai/providers`
  - ✅ **Support all 3**: OpenAI, Claude, Gemini (already configured!)
  - ❌ **NEVER do**: `const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY })`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TokenRollAI/fast-mvp](https://github.com/TokenRollAI/fast-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
