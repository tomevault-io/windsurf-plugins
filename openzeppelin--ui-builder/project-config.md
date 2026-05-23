---
trigger: always_on
description: Use this rules set if you are working directly on the app package functionality or need to learn more about the builder app package.
---


# UI Builder: App Package Rules (@app)

## 1. Overview

**Purpose:** The `packages/builder` package (`@openzeppelin/ui-builder-app`) is the main application hub for the UI Builder. It encompasses the primary user interface for constructing forms, the central **chain‑agnostic** logic, and the export system that generates standalone, runnable web application projects.

**Key Role:** Acts as the orchestrator, integrating functionalities from `@openzeppelin/ui-builder-renderer`, `@openzeppelin/ui-builder-styles`, `@openzeppelin/ui-builder-react-core`, and the various adapter packages (e.g., `@openzeppelin/ui-builder-adapter-evm`). It manages the application state (e.g., the multi‑step builder wizard) and delivers the primary user‑facing experience and export capabilities.

## 2. Core Functionality

- **Form Builder UI:** (`src/components/UIBuilder/`) Contains the multi‑step wizard and components allowing users to select a blockchain/contract/function, customize generated form fields (labels, validation, visibility), and preview the form in real time using the renderer package.
- **Chain‑Agnostic Core:** (`src/core/`) Houses foundational logic independent of specific blockchains. This includes:
  - Shared TypeScript types live in `@openzeppelin/ui-builder-types` (single source of truth). The `ContractAdapter` interface is defined at `packages/types/src/adapters/base.ts` and implemented by adapter packages.
  - Utility functions: Prefer `@openzeppelin/ui-builder-utils` (e.g., `logger`, `AppConfigService`, `cn`) over ad‑hoc implementations.
  - Reusable React providers/hooks are in `@openzeppelin/ui-builder-react-core`; component‑specific hooks reside with their components.
- **Export System:** (`src/export/`) Generates a complete, standalone React + Vite project from the user's configuration. Includes:
  - `AdapterConfigLoader`: Loads adapter‑specific configuration for export.
  - `TemplateManager`: Manages base project templates (e.g., `typescript-react-vite`) under `src/export/templates/`.
  - `FormCodeGenerator` & `TemplateProcessor`: Use templates in `src/export/codeTemplates/` and the render schema to create app code.
  - `PackageManager`: Resolves runtime/dev dependencies for the exported project based on template, adapter, and fields.
  - `StyleManager`: Ensures required styling and theme files are included.
  - `ZipGenerator`: Produces a downloadable `.zip` using `jszip`, entirely in the browser.
  - `cli/`: Export CLI at `src/export/cli/export-app.cjs`; use via `pnpm export-app export`.

## 3. Architecture

- **Monorepo Context:** Situated within the `ui-builder` pnpm monorepo. It is governed by root configurations (ESLint, Prettier, TypeScript, Vitest).
- **Package Dependencies:**
  - Consumes `@openzeppelin/ui-builder-renderer` for displaying form previews and as a core dependency in exported projects.
  - Consumes `@openzeppelin/ui-builder-styles` for the base theme and styles.
  - Consumes various `@openzeppelin/ui-builder-adapter-*` packages for blockchain‑specific functionality.
  - Consumes `@openzeppelin/ui-builder-react-core` for shared providers/hooks and context.
- **Adapter Pattern:** This is a foundational design principle. The `ContractAdapter` interface is defined in `packages/types/src/adapters/base.ts`, but the concrete implementations reside in separate packages (e.g., `@openzeppelin/ui-builder-adapter-evm`). These external adapters _must_ strictly adhere to the `ContractAdapter` interface contract. Compliance is verified by the `lint:adapters` script and CI pipeline.
- **Styling:**
  - Imports base styles and CSS variables from `@styles/global.css`.
  - Leverages Tailwind utility classes extensively for component styling.
  - Consumes centralized root configs (`tailwind.config.cjs`, `postcss.config.cjs`, `components.json`) via lightweight JS proxy files. **Does not** define its own theme; it inherits the centralized theme.
- **State Management:** Primarily relies on standard React Hooks (`useState`, `useReducer`, `useContext`). The Context API is used for managing state across the multi-step form builder wizard.
  - **State Management:** Uses modular React hooks and providers (see `@openzeppelin/ui-builder-react-core` for adapter/wallet context) and focused builder hooks under `src/components/UIBuilder/hooks` for the multi-step wizard.
  - **Export System Flow:** User Interaction -> UI State (`BuilderFormConfig`) -> `FormSchemaFactory` (using an external Adapter) -> Render schema -> `AppExportSystem` (orchestrates `AppCodeGenerator`, `TemplateManager`, `PackageManager`, `StyleManager`) -> `ZipGenerator` -> Download.
  - **Vite Build & Plugins:** Uses Vite for the development server and production builds (`vite.config.ts`). The builder defines virtual modules to reliably load cross‑package assets/config and uses `import.meta.glob` for template discovery.

## 4. Coding Guidelines & Conventions

- **General:** Strictly adhere to all root-level project guidelines: Conventional Commits (`pnpm commit`), ESLint/Prettier (`pnpm fix-all`), and TypeScript best practices. Consult the root `README.md` and `tech-stack-rule.mdc`.
- **TypeScript:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenZeppelin/ui-builder](https://github.com/OpenZeppelin/ui-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
