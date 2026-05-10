---
trigger: always_on
description: > For a complete and accurate architecture reference, read [`docs/README.md`](docs/README.md) first. The sections below are a quick-start guide; `docs/` is the authoritative source.
---

# Coding Agent Team - Developer Guide

> For a complete and accurate architecture reference, read [`docs/README.md`](docs/README.md) first. The sections below are a quick-start guide; `docs/` is the authoritative source.

## Project Overview

A CLI tool that coordinates git worktrees, tmux sessions, AI agents (Claude/Gemini), and GitHub PR state across one or more projects. It is more than a worktree manager — it is a workspace coordinator for multi-repo AI-assisted development.

## Architecture

### Tech Stack
- **Runtime**: Node.js 18+ (ESM modules)
- **Framework**: Ink (React for CLI)
- **Language**: TypeScript with strict mode
- **Testing**: Jest with ts-jest
- **Build**: tsc compiler

### Core Concepts

#### 1. **Worktrees** 
Git worktrees allow multiple branches to be checked out simultaneously in different directories. This app manages worktrees in a structured way:
- Main projects: `{projects-directory}/{project-name}/`
- Feature branches: `{projects-directory}/{project-name}-branches/{feature-name}/`
- Archived features: `{projects-directory}/{project-name}-archived/archived-{timestamp}_{feature-name}/`

The projects directory is configurable:
- **CLI Argument**: `devteam --dir /path/to/projects`
- **Environment Variable**: `PROJECTS_DIR=/path/to/projects devteam`
- **Default**: Current working directory

#### 2. **Tmux Sessions**
Each worktree gets associated tmux sessions:
- Main session: `dev-{project}-{feature}` (for Claude AI)
- Shell session: `dev-{project}-{feature}-shell` (for terminal work)
- Run session: `dev-{project}-{feature}-run` (for executing commands)

#### 3. **AI Tool Integration**
The app supports multiple AI CLIs (Claude, Gemini) and monitors AI status in tmux panes:
- Working: Shows "esc to interrupt"
- Waiting: Shows numbered prompt (e.g., "1. ")
- Idle: Shows standard prompt
- Thinking: Shows thinking indicator

Tool preference is stored per-project in `.devteam/config.json`.
## Project Structure

See [`docs/reference/code-map.md`](docs/reference/code-map.md) for the full file map. Key layout:

```
src/
├── App.tsx                 # Root component; provider nesting
├── bootstrap.tsx           # Ink render entry
├── bin/devteam.ts          # CLI executable
├── cores/                  # Core engine classes (business logic, no React)
│   ├── WorktreeCore.ts     # Worktree list, sessions, git status
│   └── GitHubCore.ts       # PR status, cache, GitHub operations
├── engine/core-types.ts    # CoreBase<T> interface
├── contexts/               # React wrappers around Core engines
│   ├── WorktreeContext.tsx
│   ├── GitHubContext.tsx
│   ├── UIContext.tsx        # Navigation state machine (no Core behind it)
│   └── InputFocusContext.tsx
├── screens/                # Three full-screen components
│   ├── WorktreeListScreen.tsx
│   ├── CreateFeatureScreen.tsx
│   └── ArchiveConfirmScreen.tsx
├── services/               # Stateless external I/O (git, tmux, gh, disk)
├── components/             # UI components (dialogs, views, common)
├── hooks/                  # useKeyboardShortcuts and others
├── models.ts               # WorktreeInfo, PRStatus, GitStatus, SessionInfo
└── constants.ts            # AI_TOOLS, refresh intervals

tests/
├── fakes/                  # In-memory service implementations
│   └── stores.ts           # Shared memory data stores
├── utils/renderApp.tsx     # Test app renderer
├── unit/                   # Unit tests
└── e2e/                    # E2E tests (mock-rendered + terminal/)
```

## Coding Conventions

### TypeScript/React Patterns

1. **Import Style**: Use ESM imports with `.js` extension (even for TS files)
   ```typescript
   import {GitService} from '../services/GitService.js';
   ```

2. **JSX Syntax**: Use modern JSX syntax for React components
   ```tsx
   return (
     <Box flexDirection="column">
       <Text>Hello</Text>
     </Box>
   );
   ```

3. **File Extensions**: 
   - `.ts` for non-React files (services, utils, models)
   - `.tsx` for React components and contexts

4. **Class Models**: Use classes with constructor initialization
   ```typescript
   export class WorktreeInfo {
     project: string;
     feature: string;
     constructor(init: Partial<WorktreeInfo> = {}) {
       this.project = '';
       this.feature = '';
       Object.assign(this, init);
     }
   }
   ```

5. **Service Pattern**: Services are classes with dependency injection
   ```typescript
   export class WorktreeService {
     constructor(
       private gitService?: GitService,
       private tmuxService?: TmuxService
     ) {
       this.gitService = gitService || new GitService();
       this.tmuxService = tmuxService || new TmuxService();
     }
   }
   ```

6. **Context Providers**: Subscribe to a Core engine; re-render on state change
   ```tsx
   export function WorktreeProvider({children}) {
     const [state, setState] = useState(() => core.getState());

     useEffect(() => core.subscribe(setState), []);

     const value = { ...state, createFeature: core.createFeature.bind(core) };
     return (
       <WorktreeContext.Provider value={value}>
         {children}
       </WorktreeContext.Provider>
     );
   }
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-era/devteam](https://github.com/agent-era/devteam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
