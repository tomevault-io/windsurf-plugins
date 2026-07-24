---
trigger: always_on
description: Copyright (c) Meta Platforms, Inc. and affiliates.
---

<!--
Copyright (c) Meta Platforms, Inc. and affiliates.

This source code is licensed under the MIT license found in the
LICENSE file in the root directory of this source tree.
-->

# CLAUDE.md - Developer Guide for AI Assistants

This guide provides context about the bunnylol.rs repository structure and patterns to help work efficiently on this codebase.

## Project Overview

**bunnylol.rs** is a smart bookmark server written in Rust that lets you create URL shortcuts accessible from your browser's search bar. It's a modern Rust implementation of [bunny1](https://github.com/ccheever/bunny1).

**Tech Stack:**
- **Language:** Rust (2024 edition)
- **Web Framework:** Rocket 0.5 (async)
- **Frontend:** Leptos 0.6 (SSR for bindings page)
- **CLI:** clap 4.5 with subcommands
- **Deployment:** Native services (systemd/launchd/Windows Service) or Docker (compose v2)

**Key Features:**
- Smart URL routing with command patterns (e.g., `gh username/repo` → GitHub)
- Multiple aliases per command (e.g., `ig`/`instagram`, `tw`/`twitter`)
- Subcommand support (e.g., `meta pay`, `ig reels`)
- Default Google search fallback
- Web portal to view all command bindings
- Unified CLI with command execution and server management

## Repository Structure

```
bunnylol.rs/
├── src/
│   ├── main.rs                          # CLI entry point and dispatcher
│   ├── lib.rs                           # Library exports
│   ├── config/                          # Configuration (server, aliases, history)
│   │   ├── mod.rs                       # Config schema, loading, and serialization
│   │   ├── user_bindings.rs             # [user_bindings] schema and resolution
│   │   └── alias_migration.rs           # Legacy [aliases] migration
│   ├── server/
│   │   ├── mod.rs                       # Rocket server setup and routing
│   │   ├── routes.rs                    # HTTP route handlers
│   │   └── web.rs                       # Web response helpers
│   ├── commands/
│   │   ├── mod.rs                       # Module exports
│   │   ├── github.rs                    # Example: gh command
│   │   ├── instagram.rs                 # Example: ig command with subcommands
│   │   ├── meta.rs                      # Example: meta command with subcommands
│   │   └── [30+ other command files]
│   ├── utils/
│   │   ├── bunnylol_command.rs          # Core trait & registry
│   │   └── url_encoding.rs              # URL building helpers
│   ├── components/
│   │   └── bindings_page.rs             # Leptos UI for /bindings
│   └── service_installer/               # Cross-platform service installation
│       ├── mod.rs
│       ├── installer.rs                 # Install/uninstall services
│       ├── manager.rs                   # Service management (start/stop/logs)
│       └── error.rs                     # Error types
├── Cargo.toml
├── docker-compose.yml
├── Dockerfile
├── README.md
└── CLAUDE.md (this file)
```

## Architecture Patterns

### 1. BunnylolCommand Trait

All commands implement the `BunnylolCommand` trait defined in `src/utils/bunnylol_command.rs`:

```rust
pub trait BunnylolCommand {
    const BINDINGS: &'static [&'static str];  // Command aliases
    fn process_args(args: &str) -> String;     // Returns URL
    fn get_info() -> BunnylolCommandInfo;      // For documentation
}
```

### 2. Command Registration

Commands are registered in two places:

1. **`src/commands/mod.rs`** - Module exports:
   ```rust
   pub use self::github::GitHubCommand;
   pub use self::instagram::InstagramCommand;
   // ... etc
   ```

2. **`src/utils/bunnylol_command.rs`** - In `BunnylolCommandRegistry`:
   - `process_command()` method (~line 74-108): Routes commands to handlers
   - `get_all_commands()` method (~line 112-148): Lists all commands for /bindings page

### 3. URL Building Helpers

Located in `src/utils/url_encoding.rs`:
- `build_search_url(base, param, query)` - Constructs search URLs with encoded params
- `build_path_url(base, path)` - Appends path to base URL

## User-Defined Bindings (`[user_bindings]`)

Users can add personal shortcuts **without recompiling** via the
`[user_bindings]` table in `~/.config/bunnylol/config.toml`. Every entry uses
the inline-table form — short-string form (`cal = "..."`) is rejected by the
parser:

```toml
[user_bindings]
# URL binding: maps a name to a URL. {} is a placeholder for URL-encoded args.
cal  = { url = "https://calendar.google.com/calendar/u/1/r" }
jira = { url = "https://corp.atlassian.net/browse/{}", description = "Jira ticket" }

# Command binding: rewrites to another bunnylol command.
work = { command = "gh mycompany/repo", description = "Work repo" }

# Override a built-in (off by default).
gh   = { command = "gh myorg/myrepo", override = true }
```

**Two variants of `UserBinding` (defined in `src/config/user_bindings.rs`):**
- `Url { url, description?, override? }` — `{}` template substitution; arg-less
  static URLs ignore extra args after the binding name.
- `Command { command, description?, override? }` — rewrites the input verbatim
  and dispatches into the registry **exactly once**. No `{}` substitution.
  Extra args are dropped. Never recurses into another `[user_bindings]` entry.

**Resolution order in `BunnylolCommandRegistry::process_command`:**
1. Prefix handlers (`$TICKER`, `r/sub`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/bunnylol.rs](https://github.com/facebook/bunnylol.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
