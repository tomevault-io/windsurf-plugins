---
trigger: always_on
description: This is a task management plugin for SiYuan Notes, developed to practice the Bullet Journal method. The plugin supports document and block reminders, calendar view for scheduling, Pomodoro timer for focus, project kanban boards, Eisenhower Matrix view, and more.
---

# GitHub Copilot Instructions for SiYuan Task Note Management Plugin

## Project Overview

This is a task management plugin for SiYuan Notes, developed to practice the Bullet Journal method. The plugin supports document and block reminders, calendar view for scheduling, Pomodoro timer for focus, project kanban boards, Eisenhower Matrix view, and more.

**Key Philosophy**: This plugin is built around the "Bullet Journal Method" (防弹笔记法), focusing on task-centered note-taking rather than just material organization. Each note should revolve around a specific task with clear action purposes and steps.

## Technology Stack

- **Language**: TypeScript
- **UI Framework**: Svelte
- **Build Tool**: Vite
- **Plugin API**: SiYuan Plugin API
- **Date/Time Parsing**: chrono-node
- **Charts**: ECharts
- **Internationalization**: Custom YAML-based i18n system

## Project Structure

```
src/
├── index.ts              # Main plugin entry point
├── api.ts                # API functions for SiYuan integration
├── components/           # UI components (dialogs, panels, views)
├── libs/                 # Utility libraries
├── types/                # TypeScript type definitions
├── utils/                # Helper utilities
├── SettingPanel.svelte   # Settings UI component
└── index.scss            # Main styles

public/i18n/              # Internationalization files
├── en_US.json
├── zh_CN.json
└── README.md

scripts/                  # Development and build scripts
├── make_dev_copy.js      # Copy plugin to SiYuan for development
├── make_dev_link.js      # Create symbolic links for development
├── make_install.js       # Install plugin to SiYuan
└── utils.js              # Script utilities
```

## Coding Conventions

### TypeScript/JavaScript

1. **File Headers**: Include copyright headers with author, date, and description:
   ```typescript
   /*
    * Copyright (c) 2024 by [author]. All Rights Reserved.
    * @Author       : [author]
    * @Date         : [date]
    * @FilePath     : /path/to/file
    * @LastEditTime : [date]
    * @Description  : [description]
    */
   ```

2. **Naming Conventions**:
   - Classes: PascalCase (e.g., `ReminderDialog`, `ProjectPanel`)
   - Functions/Methods: camelCase (e.g., `loadSettings`, `createDocWithMd`)
   - Constants: UPPER_SNAKE_CASE (e.g., `STORAGE_NAME`, `TAB_TYPE`)
   - Private class members: prefix with underscore or use TypeScript private modifier

3. **Code Style**:
   - Use async/await for asynchronous operations
   - Prefer const over let when variables won't be reassigned
   - Use descriptive variable names, especially for Chinese contexts (可以使用中文注释)

4. **Error Handling**:
   - Always use try-catch for async operations
   - Show user-friendly error messages using `showMessage()`
   - Log detailed errors to console for debugging

### Svelte Components

1. Use TypeScript in Svelte files with `<script lang="ts">`
2. Follow reactive declarations pattern with `$:` for computed values
3. Use proper component lifecycle hooks (onMount, onDestroy)
4. Keep components focused and single-responsibility

### Internationalization

1. **Always support both English and Chinese**:
   - Use the `i18n()` function from `src/pluginInstance.ts` for all user-facing strings
   - Add translations to both `public/i18n/en_US.json` and `public/i18n/zh_CN.json`
   - Translation keys should be descriptive camelCase

2. **Example usage**:
   ```typescript
   import { i18n } from "./pluginInstance";
   showMessage(i18n("taskCreatedSuccessfully"));
   ```

### API Integration

1. **SiYuan API Usage**:
   - Import API functions from `src/api.ts`
   - Common operations: `createDocWithMd`, `getBlock`, `updateBlock`, `setBlockAttrs`
   - Always handle API errors gracefully

2. **Block and Document Operations**:
   - Use block IDs (22-character strings) to reference blocks
   - Use notebook IDs and paths for document operations
   - Support block references in format: `((blockId "title"))`
   - Support SiYuan links in format: `siyuan://blocks/blockId`

### Data Management

1. **Storage**:
   - Use `plugin.loadData(STORAGE_NAME)` for persistent storage
   - Settings stored in `reminder-settings.json`
   - Reminder data in `.siyuan/storage/reminder-data.json`
   - Project data in `.siyuan/storage/project-data.json`

2. **Date/Time Handling**:
   - Use local date/time strings in format: "YYYY-MM-DD HH:mm"
   - Leverage chrono-node for natural language date parsing
   - Helper functions in `src/utils/dateUtils.ts`

## Development Workflow

### Setup

```bash
# Install dependencies
npm install
# or
pnpm install

# Development mode (with hot reload)
npm run dev

```

### Development Mode

1. **Auto-copy to SiYuan**: The vite config includes auto-copy functionality in dev mode
2. **Scripts**: Use `scripts/make_dev_copy.js` to manually copy to SiYuan plugins directory
3. **Testing**: Test changes directly in SiYuan Notes application

### Build Process

1. Vite builds TypeScript and Svelte to `dist/` or `dev/` directory
2. Static assets (README, plugin.json, icons, audios) copied to output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Achuan-2/siyuan-plugin-task-note-management](https://github.com/Achuan-2/siyuan-plugin-task-note-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
