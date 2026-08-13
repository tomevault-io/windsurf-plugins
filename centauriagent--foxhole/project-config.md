---
trigger: always_on
description: This project is a Nostr client application built with React 19.x, TailwindCSS 4.x, Vite, shadcn/ui, and Nostrify.
---

# Project Overview

This project is a Nostr client application built with React 19.x, TailwindCSS 4.x, Vite, shadcn/ui, and Nostrify.

## Technology Stack

- **React 19.x**: hooks, concurrent rendering, ref-as-prop
- **TailwindCSS 4.x**: utility-first styling
- **Vite**: dev server and production bundler
- **shadcn/ui**: unstyled accessible components on Radix UI + Tailwind (48+ components in `@/components/ui`)
- **Nostrify** (`@nostrify/react`): Nostr protocol framework
- **React Router**: client-side routing with `BrowserRouter` and automatic scroll-to-top
- **TanStack Query**: data fetching, caching, state
- **TypeScript**: type-safe JS. **Never use the `any` type.**

## Project Structure

- `/src/components/` — UI components. `ui/` holds shadcn/ui primitives; `auth/` holds login components (`LoginArea`, `AuthDialog`, `AccountSwitcher`).
- `/src/hooks/` — custom hooks. Discover the full set with `ls src/hooks/`. Key ones: `useNostr`, `useAuthor`, `useCurrentUser`, `useNostrPublish`, `useUploadFile`, `useAppContext`, `useTheme`, `useToast`, `useLoggedInAccounts`, `useLoginActions`, `useIsMobile`.
- `/src/pages/` — page components wired into React Router (`Index`, `NotFound`, `NIP19Page`).
- `/src/lib/` — utility functions and shared logic.
- `/src/contexts/` — React context providers (`AppContext`).
- `/src/test/` — testing utilities including the `TestApp` wrapper.
- `/public/` — static assets.
- `App.tsx` — **already configured** with `QueryClientProvider`, `NostrProvider`, `UnheadProvider`, `AppProvider`, `NostrLoginProvider`. **Read before editing**; changes are rarely needed.
- `AppRouter.tsx` — React Router configuration. The catch-all `/:nip19` route handles all NIP-19 identifiers (see the `nip19-routing` skill).

**Always read an existing file before modifying it.** Never write over `App.tsx`, `AppRouter.tsx`, or `NostrProvider` without first reading their contents.

## UI Components

Components in `@/components/ui` are unstyled, accessible primitives styled with Tailwind. They follow a consistent React 19 pattern: plain function components that type props via `React.ComponentProps<...>` and forward `ref` as a normal prop (no `React.forwardRef`), tag their root with a `data-slot` attribute, merge classes with the `cn()` utility, and define variants with `class-variance-authority`. Components built on Radix import from the unified `radix-ui` package (e.g. `import { Dialog as DialogPrimitive } from "radix-ui"`), not individual `@radix-ui/react-*` packages. When you need a specific component, list the directory (`ls src/components/ui/`) or import from `@/components/ui/<name>` — all common primitives are present (buttons, inputs, dialogs, dropdowns, forms, tables, etc.).

## System Prompt Management

The assistant's behavior is defined by this file (`AGENTS.md`). Edit it directly to change guidelines — updates take effect the next session. Specialized workflows live in `/.agents/skills/` as loadable skills, discoverable through the `skill` tool.

## Nostr Protocol Integration

### When to reuse an existing NIP vs. create a new kind

1. **Always review existing NIPs first.** Use the NIP index tool, then read candidate NIPs in detail. The goal is to find the closest existing solution.
2. **Prefer extending existing NIPs** over creating custom kinds, even if it requires minor schema compromises. Custom kinds fragment the ecosystem.
3. **When existing NIPs are close but not perfect**, use the existing kind as the base and add domain-specific tags. Document extensions in `NIP.md`.
4. **Only generate a new kind** when no existing NIP covers the core functionality, the data structure is fundamentally different, or the use case needs different storage characteristics (regular/replaceable/addressable).
5. **If a tool to generate a new kind number is available, you MUST use it** — don't pick an arbitrary number.
6. **Custom kinds MUST include a NIP-31 `alt` tag** with a human-readable description.

### Kind Ranges

- **Regular** (1000 ≤ kind < 10000): stored permanently by relays. Notes, articles, etc.
- **Replaceable** (10000 ≤ kind < 20000): only the latest event per `pubkey+kind` is stored. Profile metadata, contact lists.
- **Addressable** (30000 ≤ kind < 40000): identified by `pubkey+kind+d-tag`; only the latest per combo is stored. Articles, long-form content.

Kinds below 1000 are "legacy"; their storage behavior is per-kind (e.g. kind 1 is regular, kind 3 is replaceable).

### Tag Design Principles

- **Kind = schema, tags = semantics.** Don't create new kinds just to represent a different category of the same data.
- **Relays only index single-letter tags.** Use `t` for categories so filters like `'#t': ['electronics']` work at the relay level. Multi-letter tags (`product_type`, etc.) force inefficient client-side filtering.
- **Filter at the relay.** Pass tag filters in the query rather than fetching everything and filtering in JS.
- **For community/niche apps**, tag events with a `t` and query by it: `createEvent({ kind: 1, content, tags: [['t', 'farming']] })`, then `nostr.query([{ kinds: [1], '#t': ['farming'] }])`. Don't do this for generic platforms.

### Content Field Design


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CentauriAgent/foxhole](https://github.com/CentauriAgent/foxhole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
