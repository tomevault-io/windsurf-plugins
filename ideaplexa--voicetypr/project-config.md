---
trigger: always_on
description: macOS desktop app for offline voice transcription using Whisper AI. Built with Tauri v2 (Rust backend) and React 19 (TypeScript frontend). Features system-wide hotkey recording, automatic text insertion at cursor, local model management, and **remote transcription via network sharing**.
---

# Voicetypr

macOS desktop app for offline voice transcription using Whisper AI. Built with Tauri v2 (Rust backend) and React 19 (TypeScript frontend). Features system-wide hotkey recording, automatic text insertion at cursor, local model management, and **remote transcription via network sharing**.

## Core Commands

```bash
# Development
pnpm dev              # Frontend only (Vite)
pnpm tauri:dev        # Full Tauri app (frontend + Rust)

# Quality checks (run before commits)
pnpm lint             # ESLint
pnpm typecheck        # TypeScript compiler
pnpm test             # Vitest frontend tests
pnpm test:backend     # Rust tests (cd src-tauri && cargo test)
pnpm check            # All checks in one script

# Build
pnpm build            # Frontend build
pnpm tauri build      # Native .app bundle
```

## Project Layout

```
src/                          # React frontend
├── components/               # UI components
│   ├── ui/                   # shadcn/ui primitives
│   ├── tabs/                 # Tab panel components
│   └── sections/             # Page sections
├── contexts/                 # React context providers
├── hooks/                    # Custom React hooks
├── lib/                      # Shared utilities
├── utils/                    # Helper functions
├── services/                 # External service integrations
├── state/                    # State management (Zustand)
└── test/                     # Integration tests

src-tauri/src/                # Rust backend
├── commands/                 # Tauri command handlers
├── audio/                    # CoreAudio recording
├── whisper/                  # Transcription engine
├── remote/                   # Network sharing (server + client)
│   ├── server.rs             # HTTP server (warp)
│   ├── client.rs             # HTTP client for remote transcription
│   ├── lifecycle.rs          # Server start/stop management
│   └── settings.rs           # Saved connections persistence
├── menu/                     # System tray menu
├── ai/                       # AI model management
├── parakeet/                 # Parakeet sidecar integration
├── state/                    # Backend state management
├── utils/                    # Rust utilities
└── tests/                    # Rust unit tests
```

## Development Patterns

### Frontend
- **Framework**: React 19 with function components + hooks
- **Styling**: Tailwind CSS v4; use `@/*` path alias for imports
- **Components**: shadcn/ui in `src/components/ui/`; extend, don't modify
- **State**: React hooks + Zustand + Tauri events
- **Types**: Strict TypeScript; avoid `any`
- **Tests**: Vitest + React Testing Library; test user behavior, not implementation

### Backend
- **Language**: Rust 2021 edition
- **Framework**: Tauri v2 with async commands
- **Modules**: Commands in `commands/`; domain logic in dedicated modules
- **Style**: Run `cargo fmt` and `cargo clippy` before commits
- **Tests**: Unit tests in `tests/` directory; use `#[tokio::test]` for async

### Communication
- Frontend calls backend via `invoke()` from `@tauri-apps/api`
- Backend emits events via `app.emit()` or `window.emit()`
- Event coordination handled by `EventCoordinator` class

## Git Workflow

- **Commits**: Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`)
- **Pre-commit**: Run `pnpm check` or individual checks
- **Branches**: Feature branches off `main`
- **Never push** without explicit user instruction

```bash
git status                    # Always check first
git diff                      # Review changes
git add -A && git commit -m "feat: description"
```

## Planning and Release Discipline

- **Claim non-trivial work** through `plans/README.md`; put irreducible desktop/hardware checks in `plans/SMOKE.md`. `NEEDS-SMOKE` means code-frozen and unverified—not permission to re-implement it.
- **Triage before implementation**: issue reports need a reproducible path or diagnostic evidence (version, OS/hardware, logs, and expected vs. actual behavior). Do not turn an unconfirmed report into a speculative fix.
- **Keep Beta scope explicit**: collect suggestions during Beta, but do not silently add them to the active release. Only reproduced release blockers or explicitly approved low-risk fixes enter the train; defer broader product/architecture suggestions to a separate plan.
- **Re-cut after every product change**: once a beta is published, any code change requires `X.Y.Z-beta.(N+1)` and rerunning the affected smoke. A successful check against the same beta is not beta-to-beta proof.
- **Promote the tested line**: Stable is cut only after the final beta passes its required runtime matrix. Do not mix unrelated product changes between the tested final beta and Stable promotion.
- **CI is not runtime proof**: green checks establish compilation and automated contracts. Unperformed Windows/macOS hardware tests remain unchecked/`NEEDS-SMOKE` and must never be reported as passed.

## Gotchas

1. **macOS only**: Parakeet models use Apple Neural Engine; Whisper uses Metal GPU

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ideaplexa/voicetypr](https://github.com/ideaplexa/voicetypr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
