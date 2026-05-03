---
trigger: always_on
description: AI-powered terminal emulator built with Tauri 2 (Rust backend, React 19 frontend).
---

AI-powered terminal emulator built with Tauri 2 (Rust backend, React 19 frontend).

## About This Project

This is **Qbit's own codebase**. If you are Qbit, then you are the AI agent being developed here.
The system prompt you operate under is defined in `backend/crates/qbit-ai/src/system_prompt.rs`.
When working on this project, you have unique insight into how changes will affect your own behavior.

## Commands

```bash
# Development
just dev              # Full app (in current directory)
just dev ~/Code/foo   # Full app (opens in specified directory)
just dev-fe           # Frontend only (Vite on port 1420)

# Testing
just test             # All tests (frontend + Rust default-members)
just test-fe          # Frontend tests (Vitest, single run)
just test-watch       # Frontend tests (watch mode)
just test-rust        # Rust tests (cargo nextest, default-members only)
just test-rust-all    # Rust tests (all workspace crates incl. app/evals)
just test-e2e         # E2E tests (Playwright)
pnpm test:coverage    # Frontend coverage report

# Code Quality
just check            # All checks (biome + clippy + fmt + tests, full workspace)
just check-rust       # Fast Rust check (cargo check + fmt, default-members)
just lint-rust        # Rust lint (clippy + fmt, full workspace)
just fix              # Auto-fix frontend (biome --write)
just fmt              # Format all (frontend + Rust)

# Build
just build            # Production build
just build-rust       # Rust only (debug)

# Profiling
pnpm devtools         # Launch standalone React DevTools (run before `just dev`)

# Headless CLI mode
cargo build -p qbit
./target/debug/qbit --headless -e "prompt" --auto-approve
./target/debug/qbit -e "prompt"  # -e implies --headless
```

## Architecture

```
React Frontend (frontend/)
        |
        v (invoke / listen)
  Tauri Commands & Events
        |
        v
Rust Backend Workspace (backend/crates/) - 28 crates in 4 layers
    |
    Layer 4 (Application):
    +-- qbit (main crate - Tauri commands, CLI entry, runtime, history, CLI output)
    |
    Layer 3 (Domain):
    +-- qbit-ai (agent orchestration, planning, HITL, loop detection, tool policy, indexing)
    |
    Layer 2 (Infrastructure):
    +-- qbit-artifacts (artifact management)
    +-- qbit-context (token budget, context pruning)
    +-- qbit-evals (evaluation framework)
    +-- qbit-llm-providers (provider configuration types)
    +-- qbit-mcp (MCP client for external tool servers)
    +-- qbit-pty (terminal sessions)
    +-- qbit-session (conversation persistence)
    +-- qbit-settings (TOML config management)
    +-- qbit-shell-exec (shell execution)
    +-- qbit-sidecar (context capture)
    +-- qbit-sub-agents (sub-agent definitions and execution)
    +-- qbit-synthesis (session synthesis)
    +-- qbit-tools (tool system, registry, file ops, directory ops, AST search)
    +-- qbit-udiff (unified diff system)
    +-- qbit-web (web search, content fetching)
    +-- qbit-workflow (graph-based multi-step tasks)
    +-- rig-anthropic-vertex (Vertex AI Anthropic provider)
    +-- rig-gemini-vertex (Vertex AI Gemini provider)
    +-- rig-zai (Z.AI GLM provider)
    +-- rig-zai-anthropic (Z.AI Anthropic SSE transformer)
    +-- rig-openai-responses (OpenAI Responses adapter with explicit reasoning/text stream separation)
    |
    Layer 1 (Foundation):
    +-- qbit-core (zero internal deps)
```

## Project Structure

```
frontend/                 # React frontend
  components/
    ui/                   # shadcn/ui primitives (modify via shadcn CLI only)
    AgentChat/            # AI chat UI (messages, tool cards, approval dialogs)
    CommandBlock/         # Command history block display
    CommandPalette/       # Command palette/fuzzy finder
    DiffView/             # Unified diff visualization
    HomeView/             # Home tab view (projects list, recent directories)
      HomeView.tsx        # Main view component
      NewWorktreeModal.tsx # Modal for creating new git worktrees
      SetupProjectModal.tsx # Modal for adding new projects
    ImageModal/           # Modal for viewing expanded image attachments
    PaneContainer/        # Split pane layout system
      PaneLeaf.tsx        # Individual pane content (uses portal targets for Terminals)
    InlineTaskPlan/       # Task plan row above input
    PlanProgress/         # Alternate task plan progress visualization (not currently wired)
    SlashCommandPopup/    # Slash command popup (prompts + skills)
    SessionBrowser/       # Session management UI
    Settings/             # Settings dialog (AI, Terminal, Codebases, Advanced)
    Sidecar/              # Context capture panel
    NotificationWidget/   # Notification badge and popup (rendered in TabBar)
    StatusBar/            # Status bar component (not currently rendered in App)
    TabBar/               # Tab bar header with notifications
    Terminal/             # xterm.js terminal component with fullterm mode
      TerminalLayer.tsx   # Renders all Terminals via React portals for state persistence
    ThinkingBlock/        # Extended thinking display
    ToolCallDisplay/      # Tool execution display
    UdiffResultBlock/     # Unified diff result block

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qbit-ai/qbit](https://github.com/qbit-ai/qbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
