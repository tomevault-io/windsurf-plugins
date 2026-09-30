---
trigger: always_on
description: > React 19 admin dashboard template · Vite · Tailwind CSS v4 · react-i18next · ApexCharts · FullCalendar · Swiper
---

# AGENTS.md — TailAdmin React Pro

> React 19 admin dashboard template · Vite · Tailwind CSS v4 · react-i18next · ApexCharts · FullCalendar · Swiper

## Repo Map

```
src/
├── App.tsx                    # root router — all routes registered here
├── main.tsx                   # entry point: providers + CSS imports
├── index.css                  # Tailwind v4 @theme tokens, @utility classes, third-party overrides
├── pages/                     # route-level components (one file or folder per page)
│   ├── Dashboard/             # dashboard variants: Ecommerce, Analytics, CRM, Sales, Finance…
│   ├── AuthPages/             # SignIn, SignUp, ResetPassword, TwoStepVerification
│   ├── Ecommerce/             # ProductList, AddProduct, Billing, Invoices, Transactions…
│   ├── Forms/                 # FormElements, FormLayout
│   ├── Tables/                # BasicTables, DataTables
│   ├── Charts/                # LineChart, BarChart, PieChart, RadarChart, RadialChart
│   ├── UiElements/            # Alerts, Badges, Buttons, Modals, Tabs, Tooltips…
│   ├── Task/                  # TaskKanban, TaskList
│   ├── Email/                 # EmailInbox, EmailDetails
│   ├── Maps/                  # Maps, VectorMap
│   ├── Ai/                    # AI generator pages (Text, Image, Code, Video) + AiSettings
│   ├── Layouts/               # LayoutOne … LayoutSix (alternative sidebar demos)
│   └── OtherPage/             # NotFound, ComingSoon, Maintenance, Success, 500, 503…
├── components/
│   ├── ui/                    # primitives: alert/, avatar/, badge/, button/, card/,
│   │                          #   carousel/, dropdown/, modal/, pagination/, table/, tabs/,
│   │                          #   tooltip/, popover/, progressbar/, spinner/, ribbons/…
│   ├── form/                  # Form, Label, Select, MultiSelect, date-picker + input/, switch/
│   ├── common/                # shared widgets: PageBreadCrumb, ComponentCard, PageMeta,
│   │                          #   ThemeToggleButton, ScrollToTop, TableDropdown, ChartTab…
│   ├── header/                # AppHeader dropdowns (notifications, user menu, language…)
│   └── <feature>/             # one folder per domain: ecommerce/, crm/, analytics/,
│                              #   charts/, chats/, task/, invoice/, ai/, maps/…
├── layout/                    # AppLayout (sidebar+header shell), AlternativeLayout,
│                              #   AppSidebar, AppHeader, Backdrop, SidebarWidget
├── context/                   # ThemeContext, SidebarContext, LanguageContext
├── hooks/                     # useModal, useGoBack, useClickOutside
├── i18n/                      # index.ts — i18next bootstrap (resources, defaultNS, fallbackLng)
├── locales/                   # translation files: en/common.json, ar/common.json,
│                              #   es/common.json, de/common.json
├── icons/                     # .svg files + index.ts barrel (SVGR named exports)
└── utils/                     # utility helpers
```

## Stack

- **React 19** with strict **TypeScript** (~5.7), bundled by **Vite 8**.
- **React Router v7** (`react-router`) for client-side routing via `BrowserRouter`.
- **react-i18next v17** + **i18next v26** for internationalization and RTL support.
- **Tailwind CSS v4** — configured entirely through `src/index.css`; **no `tailwind.config` file exists**.
- **react-apexcharts** for charts, **@fullcalendar/react** for the calendar, **Swiper** for carousels.
- **react-helmet-async** (`PageMeta`) for per-page `<title>` and `<meta description>`.
- Path alias: `@/*` → `src/*` (configured in `tsconfig.app.json` + `vite.config.ts`).
- Scripts: `npm run dev` (Vite dev server), `npm run build` (tsc + Vite), `npm run lint`.
- Node >= 20.19.0 || >= 22.12.0 required (Vite 8 requirement).

## Routing Conventions

- All routes are registered in `src/App.tsx` using `<Routes>` / `<Route>`.
- Three layout groups:
  - **`<AppLayout>`** — standard dashboard shell (sidebar + header). Most pages live here.
  - **`<AlternativeLayout>`** — full-width shell for AI generator pages and AI settings.
  - **No layout** — standalone pages: auth routes (`/signin`, `/signup`…), error pages, layout demos.
- **New page** → create a file or folder under `src/pages/<Category>/MyPage.tsx`, then add a `<Route>` in `App.tsx` under the appropriate layout group.
- Page files are **PascalCase** (`MyPage.tsx`) with a **default export**.
- Colocate route-only sub-components inside the page folder. Reusable UI goes in `src/components/<feature>/`.

## Conventions

- **Component files**: PascalCase (`EcommerceMetrics.tsx`) with a **default export**.
- **Hook files**: camelCase (`useModal.ts`).
- **New reusable component** → `src/components/<feature>/` if domain-specific, else `src/components/common/` or `src/components/ui/`.
- **New icon** → drop the `.svg` into `src/icons/`, add a named export to `src/icons/index.ts` using a PascalCase name (e.g., `export { ReactComponent as MyIcon } from "./my-icon.svg"`). Never inline SVG markup in components.
- **Page meta (SEO)** → every page must render `<PageMeta title="…" description="…" />` (from `src/components/common/PageMeta.tsx`) as the first child.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TailAdmin/free-react-tailwind-admin-dashboard](https://github.com/TailAdmin/free-react-tailwind-admin-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
