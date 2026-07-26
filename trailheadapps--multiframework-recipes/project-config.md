---
trigger: always_on
description: This is a **Salesforce DX project** containing a **React app** that serves as a sample "recipes" repository -- like [lwc-recipes](https://github.com/trailheadapps/lwc-recipes) but for React on Salesforce.
---

# React Recipes -- Developer & Agent Guide

This is a **Salesforce DX project** containing a **React app** that serves as a sample "recipes" repository -- like [lwc-recipes](https://github.com/trailheadapps/lwc-recipes) but for React on Salesforce.

**Goal:** Teach things a React developer doesn't know about Salesforce, and things a Salesforce developer doesn't know about React -- at the intersection.

**Non-goal:** Teach React fundamentals.

The full project plan lives in **PLAN.md**. This file covers what you need to know to contribute.

---

## Project Structure

```
multiframework-recipes/                     # Repository root (Salesforce DX project)
  sfdx-project.json                         # Salesforce DX config (API v67.0)
  package.json                              # Root -- SFDX scripts only, NOT the React app
  data/                                     # Sample data for sf data import tree (planned)
  force-app/main/default/                        # Shared metadata (objects, classes, etc.)
  force-app/main/react-recipes/
    uiBundles/reactRecipes/                        # <-- THE REACT APP (all work happens here)
      src/
        app.tsx                             # React entry: Router + providers
        appLayout.tsx                       # Shell: Navbar + <Outlet />
        routes.tsx                          # Route definitions (React Router 7)
        api/
          graphqlClient.ts                  # executeGraphQL wrapper
          graphql-operations-types.ts       # Generated types (from codegen)
          utils/
            query/                          # .graphql read queries
            mutation/                       # .graphql write mutations
        recipes/                            # Individual recipe components (planned)
          hello/
          read-data/
          modify-data/
          salesforce-apis/
          error-handling/
          routing/
          styling/
        components/
          app/                              # App shell components (Navbar, Layout, CodeBlock)
          recipe/                           # Shared recipe UI (Skeleton, ContactTile, etc.)
          ui/                               # shadcn/ui primitives
        pages/                              # Route pages -- containers that render recipes
        lib/                                # Utilities (cn, etc.)
        styles/                             # global.css, slds.css
        assets/                             # Icons, images
      scripts/
        get-graphql-schema.mjs              # GraphQL introspection from connected org
      codegen.yml                           # GraphQL codegen config
      ui-bundle.json                   # Salesforce routing (SPA fallback)
      reactRecipes.uibundle-meta.xml         # Salesforce metadata
      package.json                          # React app dependencies
      vite.config.ts                        # Vite 7 + Tailwind + Salesforce plugin
      tsconfig.json                         # TypeScript strict mode
      dist/                                 # Build output (deployed to Salesforce)
```

**Key point:** There are **two `package.json` files**. The root one is for SFDX metadata scripts. The one inside `reactRecipes/` is for the React app. Almost all work happens in the React app directory.

---

## Commands

All React app commands run from `force-app/main/react-recipes/uiBundles/reactRecipes/`:

```bash
cd force-app/main/react-recipes/uiBundles/reactRecipes

npm run start              # Vite dev server
npm run build              # tsc -b && vite build (MUST pass before finishing work)
npm run lint               # ESLint (MUST pass before finishing work)
npm run test               # Vitest
npm run graphql:schema     # Fetch GraphQL schema from connected org
npm run graphql:codegen    # Generate TypeScript types from .graphql files
```

### Deploying

From the **project root**:

```bash
# Build first
cd force-app/main/react-recipes/uiBundles/reactRecipes && npm i && npm run build && cd -

# Deploy to org
sf project deploy start --source-dir force-app/main/react-recipes/uiBundles/reactRecipes --target-org <alias>
```

### Seeding Sample Data

```bash
sf data import tree --plan data/data-plan.json --target-org <alias>
```

---

## Tech Stack

| Concern           | Tool                                                                |
| ----------------- | ------------------------------------------------------------------- |
| Framework         | React 19                                                            |
| Routing           | React Router 7 (`react-router`, not `react-router-dom`)             |
| Build             | Vite 7 + `@salesforce/vite-plugin-ui-bundle`                        |
| App shell styling | Tailwind CSS 4 + shadcn/ui                                          |
| SLDS styling      | `@salesforce-ux/design-system` 2.29.0 (CSS classes on JSX)          |
| Data (GraphQL)    | `executeGraphQL` via `@salesforce/sdk-data`                         |
| Data (REST)       | `await createDataSDK()` then `.fetch()` from `@salesforce/sdk-data` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trailheadapps/multiframework-recipes](https://github.com/trailheadapps/multiframework-recipes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
