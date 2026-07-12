---
trigger: always_on
description: Guidance for Claude Code and other AI assistants working in this repository.
---

# AGENTS.md

Guidance for Claude Code and other AI assistants working in this repository.

**TeamPilot** is a Flutter client (`client/`, package `teampilot`, data ID `com.hhoa.teampilot`) that manages **workspaces**, **team launch identities**, sessions, skills, plugins, and extensions, and embeds terminals running AI agent CLIs (local PTY on desktop, or SSH — always on Android, optional on desktop). The home UI is an Apifox-style workspace shell with a built-in IDE (file tree, editor, Git, worktrees).

| Docs | Purpose |
|------|---------|
| [README.md](README.md) (English) / [README.zh.md](README.zh.md) (简体中文) | User-facing |
| [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) | Clone, commands, tests, packaging, CI |
| [docs/CODE_QUALITY.md](docs/CODE_QUALITY.md) | File size, layering, tests, Extension conventions |
| [docs/DEBUGGING.md](docs/DEBUGGING.md) | Systematic debugging process |
| [docs/PERFORMANCE_ANALYSIS.md](docs/PERFORMANCE_ANALYSIS.md) | DevTools performance JSON offline analysis (`tool/analyze_performance_json.dart`) |
| [docs/workspace-storage-layout.md](docs/workspace-storage-layout.md) | On-disk layout under `<teampilotRoot>` |

All app code lives under `client/lib/` (cubits, pages, repositories, services, models). Vendored deps: `client/packages/` (git submodules: xterm, flutter_pty_new, dartssh2, re-editor, flutter_alacritty).

## Core concepts

| Concept | Model / cubit | Role |
|---------|---------------|------|
| **Workspace** | `Workspace`, `SessionRepository` | *Where* work happens: repo folder(s), sessions, workspace-scoped resource bindings (`project-config.json`). |
| **Launch profile** | `TeamProfile` (`LaunchProfile`), `LaunchProfileCubit` | *Who/how* to launch a **team**: roster of expert keys, `ConfigBundle`, per-tool provider/model/effort, skills/plugins/MCP. Persisted under `launch-profiles/{id}/profile.json`. |
| **Expert** | `DiscoverableMember`, `ExpertCapabilityPack` | Capability pack: persona + `skillDeps` / `pluginDeps` / `mcpDeps`. Resolved via `ExpertCapabilityResolver` into session config. |
| **Session** | `AppSession`, `ChatCubit` | One chat workbench tab; owns member terminals, TeamBus (mixed teams), and session runtime dirs. |

**Simple mode** = unteamed launch (empty `sessionTeam`) — not a launch-identity document. Optional `AppSession.expertKey` / CLI preset selects an expert pack; automations use fixed scope key `AutomationTabScope.simpleLaunchProfileId` (`simple`). **Team mode** = launch with a `TeamProfile` and one PTY per roster member.

Session config merge (skills / plugins / MCP ids):

```
SessionRuntimeBundle = merge(team > expert > workspace)
```

See [docs/superpowers/specs/2026-07-10-expert-capability-pack-design.md](docs/superpowers/specs/2026-07-10-expert-capability-pack-design.md). Team identities configure members, skills, plugins, MCP, and extensions under `/team-config`.

## Architecture

### Bootstrap flow

```
main.dart
  → AppPathsBootstrapper.init()              # Application Support → AppPaths
  → TeamPilotBootstrap / buildAppShell()
      → CliToolRegistry.builtIn()             # capability-based CLI registry
      → RuntimeContextRegistry / AppStorage.bindHome()
      → CliBootstrap(...)                     # provision cli-defaults trees
      → LaunchProfileRepository, SessionRepository, SessionLifecycleService
      → TeammateBusMcpGateway.ensureStarted()
      → LaunchProfileCubit, ChatCubit, TeamHubCubit, ExpertHubCubit,
        MemberPresenceCubit, MailboxCubit, BoardCubit,
        AutomationCubit, WorkspaceTerminalRegistry, …
  → MaterialApp.router (GoRouter)
```

### State, routing, and chat

- **State:** `flutter_bloc` cubits under `client/lib/cubits/`.
- **Routing:** `client/lib/router/app_router.dart`. **[HomeShell](client/lib/pages/home_workspace/home_workspace_shell.dart)** (`home_workspace_shell.dart`) renders the title bar + open-workspace tabs; **[HomeWorkspaceBodyStack](client/lib/pages/home_workspace/home_workspace_body_stack.dart)** keeps one alive **[WorkspacePage](client/lib/pages/home_workspace/workspace/workspace_page.dart)** per tab. Initial location is `/home-v2` (or last workspace per `LayoutCubit` / `applyWorkspaceEntryMode`).
- **Chat / workbench:** `WorkspacePage` (route `/home-v2/workspace/:workspaceId`) hosts conversations + manage panes. `ChatCubit` owns tabbed `TerminalSession`s; **[SessionLaunchService](client/lib/cubits/chat/session_launch_service.dart)** handles `requestOpenSession` / `connectWorkspaceSession` / `_scheduleMemberConnect` → `SessionLifecycleService.prepareLaunch` → `TerminalSession.connect`.
- **Workspace manage config:** `WorkspaceConfigSection` = settings / skills / plugins / mcp / extensions (`workspace_config_section.dart`). Agent + provider tiering for personal identities live on the **launch profile**, not in workspace manage.

### Terminal transport

| Mode | When | Implementation |
|------|------|----------------|
| Local PTY | Desktop default | `flutter_pty_new` → `LocalPtyTransport` |
| SSH | Android always; desktop optional | `dartssh2` → `SshPtyTransport`; remote CLI via `RemoteFlashskyaiCommandBuilder` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hhoao/teampilot](https://github.com/hhoao/teampilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
