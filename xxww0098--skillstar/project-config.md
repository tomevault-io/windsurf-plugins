---
trigger: always_on
description: SkillStar is a Tauri v2 desktop app with a React SPA frontend and Rust backend.
---

# SkillStar — Code Framework

## Architecture

SkillStar is a Tauri v2 desktop app with a React SPA frontend and Rust backend.

- Frontend calls backend via `invoke()` and Tauri events.
- Tauri commands are defined in `src-tauri/src/commands/mod.rs` (root handlers) with feature modules under `src-tauri/src/commands/`.
- Core domain logic lives in workspace crates under `crates/`; `src-tauri/src/core/` retains thin adapter stubs and Tauri-specific glue.
- Persistence is mixed storage under `~/.skillstar/`: JSON for config/project metadata plus SQLite for marketplace and translation caches.
- Skill distribution is symlink-based to keep project directories clean.

> Frontend-specific conventions live in [AGENTS-UI.md](./AGENTS-UI.md).

## Key Paths


| Purpose               | Path                       |
| --------------------- | -------------------------- |
| Data root             | `~/.skillstar/`            |
| Config files          | `~/.skillstar/config/`     |
| SQLite databases      | `~/.skillstar/db/`         |
| Runtime logs          | `~/.skillstar/logs/`       |
| Runtime state         | `~/.skillstar/state/`      |
| Hub root              | `~/.skillstar/hub/`        |
| Hub skills            | `~/.skillstar/hub/skills/` |
| Local authored skills | `~/.skillstar/hub/local/`  |
| Repo cache            | `~/.skillstar/hub/repos/`  |


## Project Structure (Condensed)

```text
SkillStar/
├── src/                           # React app
│   ├── features/                  # domain slices (components + hooks)
│   │   ├── my-skills/             # skill grid, cards, modals, install/export
│   │   ├── marketplace/           # marketplace browsing
│   │   ├── projects/              # project registration + agent config
│   │   ├── security/              # security scanning
│   │   └── settings/              # app settings sections
│   ├── hooks/                     # global hooks (useNavigation, useUpdater, useAiConfig)
│   ├── pages/                     # thin route-level shells
│   ├── components/                # ui/, layout/, shared/
│   ├── lib/                       # shared utilities
│   └── types/                     # shared TS types
├── src-tauri/
│   ├── src/
│   │   ├── commands/              # mod.rs: skills, bundles, shell, network; + marketplace, agents, projects, github, patrol, acp
│   │   │   └── ai/               # AI commands split: translate, summarize, scan
│   │   │   └── models_commands.rs # provider CRUD / health dashboard (split from models.rs)
│   │   │   └── oauth_commands.rs  # Codex/Gemini OAuth + account management
│   │   │   └── quota_commands.rs  # quota refresh / usage / speedtest
│   │   └── core/                  # thin adapters + Tauri-specific glue (heavy logic moved to crates/)
│   │       ├── infra/             # re-export stub → skillstar-infra
│   │       ├── ai/               # AI domain facade (translation_cache, re-exports ai_provider)
│   │       ├── ai_provider/       # re-export stub → skillstar-ai
│   │       ├── config/            # re-export stub → skillstar-config (proxy, github_mirror)
│   │       ├── git/              # git operations (ops, gh_manager, repo_history, source_resolver, dismissed_skills)
│   │       ├── projects/          # re-export stub → skillstar-projects (agents, sync)
│   │       ├── terminal/          # Launch Deck config + terminal_backend re-export
│   │       ├── skills/            # thin adapters over skillstar-skill-core (install, update, bundle, local, group, discover)
│   │       ├── security_scan/     # static/AI scanning, orchestrator, cache, logging
│   │       ├── marketplace_snapshot/ # local-first marketplace DB
│   │       ├── model_config/      # per-provider model config (codex, claude, gemini, opencode)
│   │       └── project_manifest/  # project manifest types, helpers, path resolution
│   ├── Cargo.toml
│   └── tauri.conf.json
├── crates/                        # workspace crates (domain logic)
│   ├── skillstar-skill-core/      # skill lifecycle (install, update, bundle, local, repo_scanner, ...)
│   ├── skillstar-marketplace-core/ # marketplace snapshot + FTS (flattened from src-tauri/crates/)
│   ├── markdown-translator/       # Markdown-aware translation pipeline (flattened from src-tauri/crates/)
│   ├── skillstar-ai/              # AI provider registry, translation, summarization
│   ├── skillstar-security-scan/   # security scan analyzers + orchestrator
│   ├── skillstar-terminal/        # Launch Deck / terminal backend
│   ├── skillstar-patrol/          # background update patrol
│   ├── skillstar-projects/        # project management + agent profiles
│   ├── skillstar-git/             # git operations wrapper
│   ├── skillstar-translation/     # traditional translation APIs (DeepL, MiniMax, MyMemory, ...)
│   ├── skillstar-model-config/    # provider model config + health/quota
│   ├── skillstar-config/          # user config (proxy, github_mirror, ACP)
│   ├── skillstar-infra/           # paths, fs_ops, db_pool, migration, error, util
│   ├── skillstar-core-types/      # shared domain types
│   ├── skillstar-commands/        # Tauri-agnostic command helpers (shell, network, launch, marketplace, ACP)
│   └── skillstar-cli/             # CLI argument parsing + entry point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xxww0098/SkillStar](https://github.com/xxww0098/SkillStar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
