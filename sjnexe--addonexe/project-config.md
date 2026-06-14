---
trigger: always_on
description: Welcome, AI Assistant (Jules)! This document provides specific guidelines and tips for working effectively on the AddonExe codebase. Please adhere to these instructions in addition to your general knowledge and the user's direct requests.
---

# Agent Instructions for AddonExe Development

Welcome, AI Assistant (Jules)! This document provides specific guidelines and tips for working effectively on the AddonExe codebase. Please adhere to these instructions in addition to your general knowledge and the user's direct requests.

## 1. Core Objective

Your primary goal is to assist users by completing coding tasks, such as solving bugs, implementing features, writing tests, and updating documentation, all while maintaining code quality, consistency, and adhering to project conventions.

## 2. Environment Setup

If you need to initialize or reset the environment, use the following commands. These handle dependencies, system tools, and repository state.

```bash
npm i -g npm@latest
npm i
npm run build || true
```

## 3. Understanding the Codebase

Before implementing changes, strive to understand the relevant parts of the codebase. Key architectural information can be found in `Docs/Development/README.md`. Pay attention to:

- **Project Structure Overview:**
    - `src/core/`: Infrastructure (Config, Logging, Events, Storage).
    - `src/features/`: Modular features (Economy, Moderation, Teleportation, etc.).
    - `packs/`: Compiled Addon files.
- **Utilizing Behavior and Resource Packs:**
    - Do not rely solely on scripts (`src/`) for features.
    - Use JSON files in `packs/behavior/` (e.g., loot tables, recipes, entities, functions) and `packs/resource/` (e.g., UI, textures) whenever possible for better performance and native integration.
    - Scripts should primarily handle complex logic, state management, and dynamic interactions that JSONs cannot cover.
- **Core Managers:** `src/core` handles cross-cutting concerns.
- **Features:** Each feature in `src/features/` should be self-contained (Manager, Config, Commands).
- **Configuration Files:**
    - `src/config.js` (or `.ts`): Main settings, feature toggles, owner/admin setup.
    - `src/core/ranksConfig.js`: Defines all ranks and their visual styles.
    - `src/core/panelLayoutConfig.js`: Defines the layout and content of the UI panels.
- **Coding Conventions:** Strictly follow guidelines in `Docs/Development/CodingStyle.md` and `Docs/Development/StandardizationGuidelines.md`.
- **Naming Conventions:**
    - The general rule for all project-specific JavaScript/TypeScript identifiers is that **any code style is allowed, but not snake_case**.
    - The use of `snake_case` (e.g., `my_variable`) or `UPPER_SNAKE_CASE` (e.g., `MY_CONSTANT`) is disallowed.
    - An exception is when interacting with native Minecraft APIs that require `snake_case` identifiers. In those cases, the required style must be used.
    - For full details, always refer to the latest `Docs/Development/CodingStyle.md` and `Docs/Development/StandardizationGuidelines.md`.

## 4. Workflow and Task Management

- **Chat-First Workflow:** The primary mode of operation is through the chat session. Tasks, plans, and execution happen dynamically here.
- **Large Tasks:** For large-scale projects requiring multiple sessions or batched work, we utilize the `Dev/tasks/` directory.
    - Break down big features into sub-tasks in `Dev/tasks/`.
    - Use separate Jules sessions to tackle these batches.

## 5. Documentation Responsibilities

- **Update Root `README.md`**: If you add significant new user-facing features or make major changes to the addon's functionality or setup, you **must** also update the main project `README.md` (located in the repository root) to reflect these changes. This keeps the primary user documentation current.
- **Update `Docs/` Folder**: For substantial feature changes or additions, relevant files in the `Docs/` folder (e.g., `FeaturesOverview.md`, `ConfigurationGuide.md`, `Commands.md`) should also be updated.
- **JSDoc/TSDoc Comments**: Adhere to the JSDoc/TSDoc standards outlined in `Docs/Development/StandardizationGuidelines.md`. Add comments for new functions (especially exported ones) and complex logic. Ensure types are accurate (now enforced by TypeScript).

## 6. Code Style and Quality

- **Adherence to Guidelines:** Strictly follow `Docs/Development/CodingStyle.md` and `Docs/Development/StandardizationGuidelines.md`.
- **TypeScript:** All Behavior Pack scripts are written in TypeScript (or JavaScript migrating to TypeScript) in the `src/` directory.
- **Build Artifacts:** Do not edit files in `packs/behavior/scripts/` directly. Always edit the source in `src/` and run `npm run build`.
- **Error Handling:** Implement robust error handling (e.g., `try...catch` blocks for risky operations, validation of inputs). Refer to `Docs/Development/StandardizationGuidelines.md` (Section 6) for detailed error logging standards.
- **Logging:** Utilize the `debugLog()` function from `core/logger.ts` for development messages. This is conditional on `config.debug` being true.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SjnExe/AddonExe](https://github.com/SjnExe/AddonExe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
