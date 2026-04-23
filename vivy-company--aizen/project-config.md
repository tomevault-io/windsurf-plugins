---
trigger: always_on
description: Aizen is a macOS developer tool for managing Git worktrees with integrated terminal, file browser, web browser, and AI agent support via the Agent Client Protocol (ACP).
---

# Aizen Project - Agent Instructions

## Project Overview

Aizen is a macOS developer tool for managing Git worktrees with integrated terminal, file browser, web browser, and AI agent support via the Agent Client Protocol (ACP).

## Architecture

### Current Direction

Aizen is migrating incrementally from top-level technical buckets toward a feature-first architecture.

Target direction:

```text
aizen/
├── App/
├── Features/
│   ├── Chat/
│   ├── Worktree/
│   ├── Repository/
│   ├── Browser/
│   ├── Files/
│   ├── Terminal/
│   ├── Workspace/
│   ├── Settings/
│   └── Search/
├── Platform/
├── Integrations/
├── Persistence/
├── Shared/
├── Assets.xcassets/
├── Resources/
└── cli/
```

Each migrated feature should use:

```text
FeatureName/
├── Domain/
├── Application/
├── Infrastructure/
├── UI/
└── Testing/
```

This is an incremental migration, not a single rewrite. Legacy folders still exist during transition, but new substantial work should prefer the target feature structure instead of deepening the old buckets.

See:
- `docs/specs/feature-first-architecture-migration-spec.md`

### Design Patterns

- MVVM: Views observe @ObservableObject models (e.g., AgentSession, ChatSessionViewModel).
- Actor model: Thread-safe services (ACPClient, Libgit2Service, XcodeBuildService).
- Delegation: Request handling (AgentFileSystemDelegate, AgentTerminalDelegate, AgentPermissionHandler).
- Domain services: Git operations split by domain (GitStatusService, GitBranchService, etc.).
- Core Data: 10 persistent entities with relationships.
- Modern concurrency: async/await, AsyncStream.

### Key Components

**Agent Client Protocol (ACP)**
- ACPClient (actor): Subprocess manager with JSON-RPC 2.0.
- ACPProcessManager: Process lifecycle management.
- ACPRequestRouter: Request/response routing.
- AgentSession (@MainActor): Observable session state wrapper.
- AgentInstaller: NPM, GitHub, Binary, UV installation methods.
- Supports Claude, Codex (OpenAI), and Gemini.

**Git Operations**
- RepositoryManager: CRUD for workspaces, repos, worktrees.
- Libgit2Service: Native libgit2 wrapper for git operations.
- Domain services: GitStatusService, GitBranchService, GitWorktreeService, GitDiffService, etc.
- GitDiffProvider + GitDiffCache: Diff fetching with caching.
- ReviewSessionManager: Code review sessions.

**Terminal Integration**
- GhosttyTerminalView: GPU-accelerated terminal with Metal rendering.
- Split pane support via TerminalSplitLayout.
- Terminal presets and session management.
- Shell integration with Ghostty resources.

**Chat Interface**
- ChatSessionView + ChatSessionViewModel: Full session UI.
- MessageBubbleView: Message rendering with markdown, code blocks.
- ToolCallView + ToolCallGroupView: Tool call visualization.
- Voice input with waveform visualization.
- File attachments and inline diffs.

**File Browser**
- FileBrowserSessionView: Tree view with file operations.
- FileContentView: File content display with syntax highlighting.

**CI/CD Integration**
- WorkflowSidebarView: GitHub Actions / GitLab CI display.
- WorkflowRunDetailView: Run details and logs.
- XcodeBuildManager: Xcode build integration.

## Development Guidelines

### Architecture Policy

- Treat Aizen as greenfield for internal architecture work, not as a legacy codebase that must preserve internal structure.
- Apply this policy to refactors, bug fixes, and new feature work.
- Prefer root-cause changes over local patches that merely fit the current shape.
- When adding something new, prefer the design that makes the system more scalable, maintainable, and coherent, even if that means reworking surrounding code.
- Do not preserve internal backward compatibility just to avoid touching call sites.
- Do not add shims, adapter layers, parallel code paths, or temporary wrappers unless there is a real external compatibility requirement.
- When a design is wrong, replace it from the ground up so the resulting code is simpler and more maintainable.
- Keep compatibility only at genuine external boundaries:
  - persisted Core Data / on-disk data
  - user-visible behavior that must intentionally remain stable
  - external protocols, CLIs, APIs, and integrations
- Remove dead code when changing systems instead of leaving legacy paths in place.
- If improving a feature or fixing a bug requires breaking internal structure to make the system better, prefer the cleaner break.

### Feature-First Migration Policy

- New substantial feature work should land in `Features/<FeatureName>/` whenever the ownership boundary is clear.
- For migrated features, keep all new code inside that feature subtree.
- Do not add new broad top-level `Manager` types unless they are truly app-global.
- Do not place non-view feature orchestration in `Views/`.
- Do not place feature-specific orchestration in `Utilities/`.
- Prefer explicit ownership names such as `Store`, `Coordinator`, `Repository`, `Registry`, or `Service` based on actual responsibility.
- `Domain` is for pure feature types and policies.
- `Application` is for feature state and orchestration.
- `Infrastructure` is for Core Data, ACP, libgit2, WebKit, filesystem, and other external integrations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vivy-company/aizen](https://github.com/vivy-company/aizen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
