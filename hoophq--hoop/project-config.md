---
trigger: always_on
description: - **React 19** + JavaScript (no TypeScript)
---

# Hoop WebApp V2 - Development Guidelines

## Stack
- **React 19** + JavaScript (no TypeScript)
- **Vite** - Build tool
- **Mantine v8** - Component library (sole styling solution, no Tailwind)
- **Zustand** - State management
- **Axios** - HTTP client
- **React Router v7** - Routing

## Commands
- Development: `npm run dev`
- Build: `npm run build`
- Lint: `npm run lint`
- Preview production: `npm run preview`

## Project Structure

```
src/
├── components/          # Presentational components (receive props, no business logic)
├── layout/              # App shell: Sidebar, Header, EmptyState
├── features/            # Complex features (e.g., CommandPalette)
├── stores/              # Zustand global stores (cross-route state)
├── services/            # Axios API calls (one file per domain)
├── hooks/               # Reusable custom hooks
├── utils/               # Pure utility functions
├── pages/               # Route-based pages (each route = folder)
│   ├── [Page]/
│   │   ├── index.jsx        # Page component
│   │   ├── <TabName>.jsx    # Tab bodies / top-level page slices (siblings of index)
│   │   ├── components/      # Reusable components scoped to this page (≥2 consumers)
│   │   ├── sections/        # Page-specific decomposition (single consumer, not "components")
│   │   ├── store.js         # Local store (only if state is page-specific)
│   │   └── [SubPage]/
│   │       └── index.jsx
├── App.jsx              # Root component + providers
├── Router.jsx           # Route definitions
└── main.jsx             # Entry point
```

## Architecture Rules

### Stores (Zustand)
- **Global stores** (`src/stores/`): State consumed by multiple pages (auth, user, resources, connections, agents, UI)
- **Local stores** (`src/pages/[Page]/store.js`): State that only exists in that specific page (form wizard steps, local filters)
- Stores access services for API calls. Components access stores for state.
- Access store state outside React with `useStore.getState()`

### Services (Axios)
- Base instance in `services/api.js` with auth interceptor and 401 handling
- One file per domain: `services/agents.js`, `services/resources.js`, etc.
- Services return axios promises. Stores handle the response.

### Components — Component Library Strategy

**Every UI primitive is wrapped.** We own every component — even a Button, a Table, a TextInput. App code never imports these directly from Mantine; it always imports from `@/components/`. This gives us:
- A single place to change the visual behaviour of any primitive across the whole app
- A Storybook-ready inventory: every component in `src/components/` is a candidate for a story
- Portability: if we ever replace Mantine, only the wrapper internals change, not the app

**How to apply this rule:**
1. Check `COMPONENTS.md` first — the component you need may already exist.
2. If it doesn't exist, create a wrapper in `src/components/[Name]/index.jsx` that imports the Mantine primitive internally and re-exports a styled, opinionated version.
3. The wrapper owns all `classNames`, `styles`, and default props. Call sites stay clean.
4. Update `COMPONENTS.md` with usage examples after creating the wrapper.

**Scope:**
- `src/components/` = Reusable across the whole app. No direct Mantine imports at call sites for any component that has a wrapper.
- `src/pages/[Page]/components/` = Reusable **within the page** (≥2 consumers, or a clear candidate to graduate to the global `src/components/`). May still import Mantine directly if no wrapper exists yet and it's too specific to generalise.
- `src/pages/[Page]/sections/` = **Single-consumer page decomposition.** A file that exists only to keep a tab/page body small. Not a "component" — don't put it in `components/`. Example: `pages/Roles/Configure/sections/ConnectionTagsEditor.jsx` is rendered by exactly one tab; it lives in `sections/`. A `components/SecretField` rendered by multiple renderers lives in `components/`.
- **Tabs / top-level page slices** live at the page root next to `index.jsx` (e.g. `pages/Roles/Configure/CredentialsTab.jsx`). They're not reusable and not decomposition — they're the page itself, sliced for readability.
- **Before creating a new component**, check `COMPONENTS.md` — it catalogs every existing component, hook, store, and service with usage examples.

**The reusability bar — what makes something a "component"?**

| Where it lives | What it is | Examples |
|---|---|---|
| `pages/[Page]/<X>.jsx` | A tab body or top-level slice of the page | `CredentialsTab.jsx`, `DetailsTab.jsx`, `ConfigureHeader.jsx` |
| `pages/[Page]/sections/` | Decomposition, single consumer, not reusable | `TestConnectionModal.jsx`, `ConnectionTagsEditor.jsx`, `MetadataFieldsInput.jsx` |
| `pages/[Page]/components/` | Reusable within the page (≥2 consumers) | `SecretField/`, `ToggleSection.jsx`, `ReviewSection.jsx` |
| `src/components/` | Reusable across the whole app | `Button`, `TextInput`, `Table` |

When something in `pages/[Page]/components/` starts getting reused outside the page, graduate it to `src/components/` (and update `COMPONENTS.md`).

### Layout
- `src/layout/` = Shared layout infrastructure (Sidebar, Header, EmptyState, Layout container)
- These are not generic reusable components, but structural elements that define the app shell


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoophq/hoop](https://github.com/hoophq/hoop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
