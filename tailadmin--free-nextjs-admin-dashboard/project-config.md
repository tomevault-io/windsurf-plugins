---
trigger: always_on
description: > Next.js admin dashboard template · Tailwind CSS v4 · next-intl · ApexCharts · FullCalendar · Swiper
---

# AGENTS.md — TailAdmin Pro

> Next.js admin dashboard template · Tailwind CSS v4 · next-intl · ApexCharts · FullCalendar · Swiper

## Repo Map

> Route groups `(name)/` are Next.js App Router organizational folders —
> they don't affect the URL path. Explore subfolders directly; this map
> stops at the level needed to orient, not to enumerate.

```
src/
├── app/                      # routes (Next.js App Router)
│   ├── [locale]/             # localized routes (next-intl)
│   │   ├── (admin)/          # dashboard shell (sidebar+header via src/layout)
│   │   │   ├── (home)/       # dashboard variants, e.g. analytics/, crm/, sales/
│   │   │   ├── (others-pages)/ # feature pages, e.g. calendar/, chat/, (forms)/
│   │   │   └── (ui-elements)/ # component demo pages (alerts, buttons, modals…)
│   │   ├── (full-width-pages)/ # no dashboard shell, e.g. (auth)/, coming-soon/
│   │   ├── (layouts-example)/ # sidebar layout variants (layout-one … six)
│   │   └── layout.tsx, not-found.tsx
│   ├── favicon.ico, globals.css
├── components/
│   ├── ui/                    # primitives: button, modal, table, tabs…
│   ├── form/                  # Form, Label, Select + input/, switch/
│   ├── common/                # shared widgets: PageBreadCrumb, ThemeToggleButton…
│   ├── header/                # header dropdowns
│   └── <feature>/             # one folder per domain: ecommerce, crm, invoice…
├── i18n/                     # routing.ts, request.ts, navigation.ts, languages.ts
├── messages/                 # translation dictionaries (en.json, ar.json, es.json, de.json)
├── layout/                    # admin shell: AppSidebar, AppHeader
├── context/                   # SidebarContext, ThemeContext
├── hooks/                     # useModal, useGoBack, useClickOutside
├── icons/                     # .svg + index.tsx barrel (SVGR)
├── proxy.ts                  # next-intl routing middleware
└── utils/
```

## Stack

- **Next.js 16** (App Router, Turbopack + webpack both configured) with **React 19** and strict **TypeScript**.
- **next-intl v4** for internationalization (i18n), localized routing, and RTL support.
- Path alias: `@/*` → `src/*`.
- Scripts: `npm run dev`, `npm run build`, `npm run lint`. Node >= 20.9.
- This Next.js version may differ from your training data — read the relevant guide in `node_modules/next/dist/docs/` before writing code and heed deprecation notices.

## Conventions

- **New page** → add a folder under the matching `src/app/[locale]/(...)/` group; colocate route-only components there. Never place pages outside `[locale]`.
- **New reusable component** → `components/<feature>/` if domain-specific, else `components/common/` or `components/ui/`.
- **New icon** → drop the `.svg` in `icons/`, export it from the `index.tsx` barrel with a PascalCase name. Never inline SVG markup in components.
- Route groups: `(admin)` is the only group with the sidebar/header shell; `(full-width-pages)` renders pages without chrome; `(layouts-example)` holds alternative sidebar layouts.
- Component files are **PascalCase** (`MonthlySalesChart.tsx`) with a **default export**; route files stay lowercase (`page.tsx`, `layout.tsx`); hooks are camelCase (`useModal.ts`).
- Root `app/[locale]/layout.tsx` is a Server Component setting up `NextIntlClientProvider`, fonts, direction (`dir="ltr"|"rtl"`), and providers. The `(admin)` shell layout and interactive UI are Client Components — add `"use client"` whenever using hooks, event handlers, or browser APIs.
- Import via the alias (`@/context/...`, `@/icons/...`, `@/i18n/...`) for cross-folder imports; relative imports are fine within a feature folder.

## Internationalization (next-intl) rules

- **Navigation & Routing**: Always import navigation primitives (`Link`, `useRouter`, `usePathname`, `redirect`) from `@/i18n/navigation`, never directly from `next/link` or `next/navigation`.
- **Routing Configuration**: Locales (`en`, `ar`, `es`, `de`) and routing settings are centralized in `src/i18n/routing.ts` (`localePrefix: "never"`).
- **Translations**:
  - In Client Components: use `useTranslations("namespace")`.
  - In Server Components: use `getTranslations("namespace")` from `next-intl/server`.
  - Organize translation keys by nested feature namespaces (e.g. `t("customers")` under `ecommerce.metrics`).
- **Translation Messages**: All translation dictionaries live in `src/messages/<locale>.json`. When adding or updating user-facing copy, maintain corresponding keys across all supported language files.
- **Static Rendering & Server Components**: Server layouts/pages inside `[locale]` must call `setRequestLocale(locale)` to enable static rendering with `generateStaticParams()`.
- **RTL Support**: Arabic (`ar`) uses RTL direction (`dir="rtl"`) via `isRtl(locale)` in `src/i18n/languages.ts`. Ensure UI components handle RTL layouts gracefully using CSS logical properties and `rtl:` variants.

## Styling rules

- Tailwind CSS **v4** — the theme lives in `src/app/globals.css` under `@theme`; there is no `tailwind.config`.
- Always use theme tokens instead of hardcoded values:
  - Colors: `brand`, `gray`, `blue-light`, `orange`, `success`, `error`, `warning` scales (`25`–`950`), plus `theme-pink-500` / `theme-purple-500`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TailAdmin/free-nextjs-admin-dashboard](https://github.com/TailAdmin/free-nextjs-admin-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
