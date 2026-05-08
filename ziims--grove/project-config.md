---
trigger: always_on
description: Guidelines for coding agents working in the Grove codebase.
---

# AGENTS.md

Guidelines for coding agents working in the Grove codebase.

## Project Overview

Grove is a terminal UI (TUI) for managing multiple Claude Code agents with git worktree isolation. Built with Rust using ratatui for the UI, tokio for async runtime, and git2 for git operations.

## Build/Lint/Test Commands

```bash
cargo build                    # Development build
cargo build --release          # Production build
cargo test                     # Run all tests
cargo test test_name           # Run a single test by name
cargo test -- --nocapture      # Run tests with output visible
cargo clippy --all-targets --all-features -- -D warnings  # Lint
cargo fmt -- --check           # Format check
cargo fmt                      # Auto-format
cargo run -- /path/to/repo     # Run the application
```

## Code Style Guidelines

### Error Handling

Use `anyhow` for error handling:

```rust
use anyhow::{Context, Result, bail};

fn load_config() -> Result<Config> {
    let content = std::fs::read_to_string(&path)
        .context("Failed to read config file")?;
    Ok(toml::from_str(&content).context("Failed to parse config")?)
}
```

### Async Patterns

```rust
#[tokio::main]
async fn main() -> Result<()> { /* ... */ }

let (action_tx, mut action_rx) = mpsc::unbounded_channel::<Action>();
tokio::spawn(async move { let _ = tx.send(Action::UpdateStatus { ... }); });
```

### Module Organization

Each module has a `mod.rs` that re-exports public items:

```rust
// src/agent/mod.rs
pub mod detector;
pub mod manager;
pub mod model;
pub use detector::detect_status;
pub use manager::AgentManager;
pub use model::{Agent, AgentStatus};
```

### Naming Conventions

- Functions/variables: `snake_case` (`select_next`, `agent_list`)
- Types/traits: `PascalCase` (`AppState`, `AgentStatus`)
- Constants: `SCREAMING_SNAKE_CASE` (`MAX_BUFFER_SIZE`)

### Serde Patterns

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Config {
    #[serde(default)]
    pub gitlab: GitLabConfig,
    #[serde(skip)]
    pub git_status: Option<GitSyncStatus>,  // Runtime-only
}
```

## Architecture Patterns

### Action-Based State Management

All state mutations go through the `Action` enum:

```rust
#[derive(Debug, Clone)]
pub enum Action {
    SelectNext,
    CreateAgent { name: String, branch: String },
    DeleteAgent { id: Uuid },
    UpdateAgentStatus { id: Uuid, status: AgentStatus },
    Quit,
}
```

### Widget Pattern

UI components follow the builder pattern:

```rust
pub struct AgentListWidget<'a> { agents: &'a [&'a Agent], selected: usize }

impl<'a> AgentListWidget<'a> {
    pub fn new(agents: &'a [&'a Agent], selected: usize) -> Self { /* ... */ }
    pub fn render(self, frame: &mut Frame, area: Rect) { /* ... */ }
}

AgentListWidget::new(&agents, selected).render(frame, area);
```

## TUI Rendering Patterns

```rust
use ratatui::layout::{Layout, Direction, Constraint};

let chunks = Layout::default()
    .direction(Direction::Vertical)
    .constraints([Constraint::Length(8), Constraint::Min(10)])
    .split(area);

let block = Block::default().title(" AGENTS ").borders(Borders::ALL);
let style = Style::default().fg(Color::Green).add_modifier(Modifier::BOLD);
```

## Git Workflow

Always run `cargo fmt` before pushing changes to ensure consistent code formatting:

```bash
cargo fmt && git add . && git commit
```

## Version System

Grove uses **semver + git hash versioning** - automatically generated at build time:

- **Format**: `{version} ({hash})` (e.g., `0.1.0 (abc1234)`)
- **Version**: Read from `Cargo.toml` (managed by release-plz)
- **Hash**: Short git commit hash for unique identification

### No Manual Updates Required

- Version comes from `Cargo.toml` `version` field
- Git hash is extracted at compile time via `build.rs`
- Each worktree/branch automatically gets its unique hash
- **Never edit version manually** - release-plz handles it

### Creating a Release

Releases are automated via release-plz:

1. Merge PRs to `main`
2. release-plz creates a PR with version bump and changelog
3. Merge the release PR
4. release-plz tags the release and publishes

The binary will show the semver from Cargo.toml plus the current commit hash.

### How It Works

1. `build.rs` runs before every compilation
2. Reads version from `CARGO_PKG_VERSION` environment variable (Cargo.toml)
3. Gets short git hash via `git rev-parse --short HEAD`
4. Combines into `{version} ({hash})` format
5. Writes to `$OUT_DIR/version.txt`, embedded in binary

## Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_status_detection() {
        assert!(matches!(detect_status("⠋ Reading..."), AgentStatus::Running));
    }
}
```

## Key Dependencies

| Crate | Purpose |
|-------|---------|
| `ratatui` | Terminal UI rendering |
| `crossterm` | Terminal events |
| `tokio` | Async runtime (rt-multi-thread, macros, sync, time) |
| `anyhow` | Error handling |
| `serde` | Serialization |
| `git2` | Git operations |

## File Structure

```
src/
├── main.rs          # Entry point, event loop
├── agent/           # Agent model, status detection
├── app/             # AppState, Config, Action enum
├── git/             # Git operations, worktree
├── gitlab/          # GitLab API client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZiiMs/Grove](https://github.com/ZiiMs/Grove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
