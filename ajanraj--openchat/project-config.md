---
trigger: always_on
description: - You are an agent - please keep going until the user's query is completely resolved, before ending your turn and yielding back to the user. Only terminate your turn when you are sure that the problem is solved.
---

## Global Rules

- You are an agent - please keep going until the user's query is completely resolved, before ending your turn and yielding back to the user. Only terminate your turn when you are sure that the problem is solved.
- If you are not sure about file content or codebase structure pertaining to the user's request, use search subagent where appropriate to gather the relevant information: do NOT guess or make up an answer.
- You MUST plan extensively before each function call, and reflect extensively on the outcomes of the previous function calls.
- Your thinking should be thorough and so it's fine if it's very long. You can think step by step before and after each action you decide to take.
- You MUST iterate and keep going until the problem is solved.
- Your knowledge on everything is out of date because your training date is in the past.
- You CANNOT successfully complete this task without using web search or context7 tool to verify your understanding of third party packages and dependencies is up to date.
- Take your time and think hard through every step - remember to check your solution rigorously and watch out for boundary cases.
- LESS COMPLEXITY IS BETTER, the fewer lines, the less complex logic the better.
- Do not assume anything. Use the docs from context7 tool.
- If there is a lint error (`bun run lint`), fix it before moving on.
- Use `agent_rules/commit.md` for commit instructions.

## Project Details

### Tech Stack

- **Framework**: TanStack Start (full-stack React framework with SSR)
- **Routing**: TanStack Router (type-safe file-based routing)
- **Data Fetching**: TanStack Query (powerful data fetching and caching)
- **Build Tool**: Vite 8 (next-generation frontend tooling)
- **Server**: Nitro v3 (universal server engine)
- **UI Library**: React 19
- **Package Manager**: Bun
- **Styling**: Tailwind CSS v4
- **Environment Variables**: `import.meta.env` for client-side (`VITE_` prefixed), `process.env` for server-side
- **AI Integration**: Vercel AI SDK v6
- **Backend**: Convex (real-time database, authentication, file storage, serverless functions)
- **Integrations**: Composio (Gmail, Calendar, Notion, GitHub, Slack, etc.)
- **State Management**: Zustand
- **Animations**: Framer Motion
- **Toast Notifications**: Sonner
- **Icons**: Phosphor Icons, Lucide React
- **UI Components**: Shadcn/UI, Radix UI primitives
- **Testing**: Vitest
- **Linting**: oxlint (with type-aware linting via tsgolint)
- **Formatting**: oxfmt
- **Type Checking**: tsgo (TypeScript native compiler)
- **Analytics**: PostHog, Vercel Analytics
- **Caching**: Upstash Redis
- **Object Storage**: Cloudflare R2 (via @convex-dev/r2)

## Code Standards

- Use TypeScript with strict mode enabled. Avoid `any` and `unknown` types. Prefer explicit types and interfaces. Do not use `@ts-ignore` or disable type checks.
- Use functional React components. Always use hooks at the top level. Do not use default exports for components or functions.
- When working on anything, try to split off components, utils, anything reusable to ensure better loading speed and less complexity.
- Follow TanStack Start and React 19 best practices.
- Use Tailwind CSS utility classes for styling. Avoid inline styles.
- Use Bun for all package management and scripts (`bun install`).
- Follow Convex guidelines in `agent_rules/convex_rules.md`.
- Use shadcn/ui components as documented. Do not modify library code directly. Prefer composition over modification. Follow guidelines in `agent_rules/shadcn.md` when creating or editing UI components.
- Use oxlint for linting and oxfmt for formatting. Run `bun run lint` and `bun run format` before committing. Do not use other linters or formatters (like ESLint or Prettier).
- Ensure accessibility: use semantic HTML, provide alt text for images, and follow accessibility guidelines in `agent_rules/design_guidelines.md`.
- Use `Icon` suffix for Phosphor React icons (e.g., `CaretIcon` not `Caret`).
- Access environment variables directly via `import.meta.env` for client-side and `process.env` for server-side code.
- Before using `useEffect`, always read https://react.dev/learn/you-might-not-need-an-effect to ensure it's actually needed. Most effects can be replaced with event handlers, useMemo, or derived state.

## Architecture Overview

### Frontend Architecture

- **TanStack Start**: Full-stack React framework with SSR, using file-based routing
- **Component Structure**:
  - `src/components/`: React components organized by feature (chat, layout, history, auth, etc.)
  - Components are functional with TypeScript interfaces for props
- **Routing**: File-based routing in `src/routes/` with `__root.tsx` shell pattern and API routes as `api.*.ts` files
- **State Management**:
  - Zustand stores in `src/lib/store/` for complex persisted state (e.g., theme editor)
  - TanStack Query + Convex via `convexQuery()` wrapper for server state
  - React Context providers in `src/providers/` for app-wide state (ChatSession, Sidebar, User)
- **Styling**: Tailwind CSS v4 with custom themes and animations

### Backend Architecture (Convex)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajanraj/OpenChat](https://github.com/ajanraj/OpenChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
