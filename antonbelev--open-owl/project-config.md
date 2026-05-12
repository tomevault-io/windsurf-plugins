---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open Owl is an Electron-based desktop application that provides a visual UI for managing Claude Code configurations. It enables users to configure subagents, skills, plugins, hooks, slash commands, and MCP servers through an intuitive interface, replacing manual JSON/YAML editing.

**Tech Stack:** Electron + React 18 + TypeScript + Vite + Zustand + Tailwind CSS

## Common Development Commands

### Development
```bash
npm run dev:electron    # Start Electron app in development mode
npm run dev            # Start Vite dev server only (for renderer testing)
```

### Building
```bash
npm run build          # Build all (renderer, main, preload) using electron-vite
npm run build:renderer # Build React frontend only (Vite standalone)
```

### Testing
```bash
npm test               # Run tests in watch mode
npm run test:unit      # Run all unit tests once
npm run test:integration # Run integration tests
npm run test:e2e       # Run Playwright E2E tests
npm run test:coverage  # Run tests with coverage report
```

### Code Quality
```bash
npm run lint           # Lint all code
npm run lint:fix       # Auto-fix linting issues
npm run format         # Format code with Prettier
npm run format:check   # Check code formatting
npm run typecheck      # Type-check all TypeScript
npm run clean          # Clean build artifacts
```

### Packaging
```bash
npm run package        # Package for current platform
npm run package:mac    # Build macOS .dmg
npm run package:win    # Build Windows installer
npm run package:linux  # Build Linux AppImage
```

## Architecture

### Three-Process Architecture

Open Owl follows Electron's multi-process architecture:

1. **Main Process** (`src/main/`) - Node.js backend that manages:
   - File system operations (reading/writing Claude configs)
   - IPC handlers for communication with renderer
   - Backend services (ClaudeService, SkillsService, etc.)
   - Claude CLI execution

2. **Renderer Process** (`src/renderer/`) - React frontend:
   - UI components and pages
   - State management with Zustand
   - React hooks for data fetching
   - Communicates with main via `window.electronAPI`

3. **Preload Script** (`src/preload/`) - Secure IPC bridge:
   - Exposes limited `electronAPI` to renderer
   - Maintains context isolation for security
   - Type-safe IPC communication

### IPC Communication Pattern

All inter-process communication follows this pattern:

1. **Define channel and types** in `src/shared/types/ipc.types.ts`:
   ```typescript
   export const IPC_CHANNELS = {
     CHECK_CLAUDE_INSTALLED: 'system:check-claude',
   };

   export interface CheckClaudeInstalledResponse {
     success: boolean;
     installed: boolean;
     version?: string;
     path?: string;
   }
   ```

2. **Create IPC handler** in `src/main/ipc/`:
   ```typescript
   ipcMain.handle(IPC_CHANNELS.CHECK_CLAUDE_INSTALLED, async () => {
     const result = await claudeService.checkInstallation();
     return { success: true, ...result };
   });
   ```

3. **Expose in preload** (`src/preload/index.ts`):
   ```typescript
   contextBridge.exposeInMainWorld('electronAPI', {
     checkClaudeInstalled: () => ipcRenderer.invoke(IPC_CHANNELS.CHECK_CLAUDE_INSTALLED),
   });
   ```

4. **Use in renderer** via React hook:
   ```typescript
   const result = await window.electronAPI.checkClaudeInstalled();
   ```

### Key Directories

- `src/main/services/` - Backend business logic (ClaudeService, SkillsService)
- `src/main/ipc/` - IPC handlers grouped by domain (systemHandlers, skillsHandlers)
- `src/renderer/components/` - React components organized by feature
- `src/renderer/hooks/` - React hooks for data fetching and state
- `src/shared/types/` - TypeScript types shared between main and renderer
- `src/shared/utils/` - Utility functions (path manipulation, validation)

## Adding New Features

Follow this end-to-end flow (see completed example: Claude Code detection feature):

1. **Define types** in `src/shared/types/` (ipc.types.ts, agent.types.ts, etc.)
2. **Create backend service** in `src/main/services/` with business logic
3. **Add IPC handlers** in `src/main/ipc/` and register in `src/main/index.ts`
4. **Update preload** in `src/preload/index.ts` to expose IPC methods
5. **Create React hook** in `src/renderer/hooks/` for data fetching
6. **Build UI component** in `src/renderer/components/`
7. **Write tests** for service, hook, and component in `tests/unit/`

## Testing Patterns

### Unit Tests
- Located in `tests/unit/`
- Use Vitest + React Testing Library
- Mock `window.electronAPI` for renderer tests
- Test hooks with `renderHook` from `@testing-library/react`

### Component Test Example
```typescript
// Mock electron API
vi.stubGlobal('electronAPI', {
  checkClaudeInstalled: vi.fn().mockResolvedValue({
    success: true,
    installed: true,
    version: '1.0.0'
  })
});

// Test component
render(<ClaudeStatusCard />);
await waitFor(() => {
  expect(screen.getByText(/installed/i)).toBeInTheDocument();
});
```

### Running Single Test
```bash
npm test -- useClaudeInstallation.test.ts  # Run specific test file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonbelev/open-owl](https://github.com/antonbelev/open-owl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
