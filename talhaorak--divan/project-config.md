---
trigger: always_on
description: This file tells AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.) how to work
---

# CLAUDE.md — AI Agent Instructions for Divan

This file tells AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.) how to work
effectively in this codebase. Read this before making any changes.

---

## What Is Divan?

Divan is a Next.js 16 (App Router) dashboard for monitoring OpenClaw AI agent workspaces.
It is a **read-mostly** app: it reads workspace files (Markdown, YAML) and OpenClaw Gateway
state, and displays them in an animated, spatial UI. There is no database.

Key entry points:
- `src/app/page.tsx` — Home page (3D scene)
- `src/app/memory/page.tsx` — Memory browser
- `src/app/tasks/page.tsx` — Goal tree + TODO
- `src/app/team/page.tsx` — Agent team view
- `src/app/cron/page.tsx` — Cron job manager
- `src/app/api/*/route.ts` — Backend API routes (server-side only)
- `src/lib/workspace.ts` — All file system operations
- `src/lib/i18n.ts` — All UI strings (Turkish + English)
- `src/contexts/LanguageContext.tsx` — i18n hook

---

## Key Patterns to Follow

### 1. i18n: ALWAYS use `useLanguage()` for UI strings

**Never hardcode user-visible strings in components.** Use the `t()` function from the
`useLanguage()` hook instead.

```tsx
// ✅ Correct
import { useLanguage } from "@/contexts/LanguageContext";

export function MyComponent() {
  const { t } = useLanguage();
  return <h1>{t("myComponent.title")}</h1>;
}

// ❌ Wrong
export function MyComponent() {
  return <h1>My Title</h1>;
}
```

When you add a new string, you **must** add the key to **both** `tr` and `en` dictionaries
in `src/lib/i18n.ts`. Keys use dot-notation namespacing:

```ts
// src/lib/i18n.ts — add to BOTH dictionaries
const tr: Translations = {
  // ...
  "myComponent.title": "Başlık",
  "myComponent.someLabel": "Bir etiket",
};

const en: Translations = {
  // ...
  "myComponent.title": "Title",
  "myComponent.someLabel": "A label",
};
```

For strings with dynamic values, use `{placeholder}` syntax:
```ts
"myComponent.count": "{n} öğe",  // TR
"myComponent.count": "{n} items", // EN
```
```tsx
t("myComponent.count", { n: items.length })
```

### 2. Components

- Pages go in `src/app/<page-name>/page.tsx` — these are **server components** by default.
  Add `"use client"` at the top only when you need hooks, state, or browser APIs.
- Shared UI components go in `src/components/`.
- Use **PascalCase** for component filenames: `AgentCard.tsx`, not `agent-card.tsx`.
- Use functional components with hooks. No class components.
- Wrap client-only logic in `useEffect` to avoid hydration errors.

### 3. API Routes

All API routes live in `src/app/api/<name>/route.ts`. They follow this pattern:

```ts
import { NextResponse } from "next/server";
import { someWorkspaceHelper } from "@/lib/workspace";

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const param = searchParams.get("param");

  const data = await someWorkspaceHelper(param ?? "");
  if (!data) return NextResponse.json({ error: "Not found" }, { status: 404 });

  return NextResponse.json({ data });
}
```

Rules for API routes:
- Keep route handlers thin. Put business logic in `src/lib/`.
- All file system operations go through `src/lib/workspace.ts` helpers — never use `fs` directly in route handlers.
- Never expose environment variables (especially `OPENCLAW_GATEWAY_TOKEN`) in responses.
- Auth tokens and workspace paths are **server-side only** — never import them in client components.

### 4. File System Access

Always use the helpers in `src/lib/workspace.ts`:

```ts
import { readWorkspaceFile, writeFileContent } from "@/lib/workspace";

// Read a workspace file (returns null if missing)
const content = await readWorkspaceFile("MEMORY.md");

// Write a workspace file (returns false on failure)
const ok = await writeFileContent("memory/note.md", "content here");
```

**Never** construct paths with string concatenation. Use `path.join()`:
```ts
// ✅
const fullPath = path.join(WORKSPACE, relativePath);

// ❌
const fullPath = WORKSPACE + "/" + relativePath;
```

The `writeFileContent()` function validates that the resolved path is within WORKSPACE
(path-traversal protection). Do not bypass this check.

### 5. Environment Variables

Access env vars only in server-side code (API routes, `src/lib/`):

```ts
// ✅ In src/lib/workspace.ts or src/app/api/*/route.ts
const workspace = process.env.OPENCLAW_WORKSPACE || path.join(os.homedir(), "clawd");

// ❌ In src/components/ or src/app/page.tsx (client-side)
const workspace = process.env.OPENCLAW_WORKSPACE; // undefined in browser
```

Available env vars:
- `OPENCLAW_WORKSPACE` — path to the OpenClaw workspace (always use this, never hardcode)
- `OPENCLAW_GATEWAY_URL` — gateway WebSocket URL
- `OPENCLAW_GATEWAY_HTTP` — gateway HTTP URL
- `OPENCLAW_GATEWAY_TOKEN` — gateway auth token (never log or expose this)

### 6. Styling

- Tailwind CSS utility classes only. Do not write custom CSS unless adding a CSS custom property to `globals.css`.
- The app is **dark-theme only**. Do not add light-mode variants.
- Colour conventions:
  - Page background: `bg-[#0a0a0f]` or `bg-black`
  - Cards/panels: `bg-[#111118]` or `bg-white/5`
  - Borders: `border-[#1e1e2e]` or `border-white/10`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talhaorak/divan](https://github.com/talhaorak/divan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
