---
trigger: always_on
description: Instructions for coding agents working on the Grove repository.
---

Instructions for coding agents working on the Grove repository.

## Project Overview

Grove is a CLI tool written in Rust that manages Git worktrees. It targets Linux, macOS, and Windows platforms.

**Key technologies:**
- Language: Rust (2021 edition)
- CLI Framework: clap (derive macros)
- Git Operations: git CLI shell-outs via `std::process::Command`
- Terminal UI: dialoguer (fuzzy-select), colored
- Serialization: serde, serde_json
- Testing: Rust's built-in test framework (`cargo test`)

## Repository Structure

```
src/
├── main.rs               # CLI entry point, clap command registration
├── models.rs             # Worktree struct, option types
├── utils.rs              # Helper functions (discovery, formatting, config)
├── commands/             # One file per CLI command
│   ├── mod.rs
│   ├── add.rs
│   ├── go.rs
│   ├── init.rs
│   ├── list.rs
│   ├── pr.rs
│   ├── prune.rs
│   ├── remove.rs
│   ├── self_update.rs
│   ├── shell_init.rs
│   └── sync.rs
└── git/
    ├── mod.rs
    └── worktree_manager.rs  # Core Git worktree operations

test/
└── integration/          # Hone integration tests

site/                     # GitHub Pages website
├── index.html            # Landing page
└── install.sh            # Installation script
```

## Development Commands

```bash
# Build debug binary
cargo build

# Build optimized release binary
cargo build --release

# Run directly in development
cargo run -- <command>

# Type check without building
cargo check

# Run all tests
cargo test

# Clean build artifacts
cargo clean
```

Always run `cargo check` and `cargo test` before committing changes.

## Updating Documentation

### README.md

The README at the repository root is the primary documentation. When updating:

1. Keep the existing section structure:
   - Features
   - Installation
   - Quick Start
   - Commands (with examples)
   - Development

2. When adding a new command:
   - Add it to the Commands section with usage syntax and examples
   - Include all flags/options with descriptions
   - Show realistic example output if helpful

3. When changing command behavior:
   - Update the corresponding command documentation
   - Update any affected examples

### Site Documentation (site/)

The `site/` directory contains the GitHub Pages website. The `site/index.html` file is a standalone HTML page with embedded CSS.

When updating site documentation:

1. **Keep README and site in sync** - The site mirrors the README content. If you update README command documentation, update `site/index.html` to match.

2. **Maintain the HTML structure** - The site uses semantic HTML sections:
   - Hero section with tagline
   - Installation section
   - Commands/usage section

3. **Test locally** - Open `site/index.html` in a browser to verify changes render correctly.

4. **Deployment** - The site auto-deploys via GitHub Actions when changes to `site/` are pushed to main.

### install.sh

The `site/install.sh` script is the curl-pipe-bash installer. When modifying:

- Test the script thoroughly on both Linux and macOS
- Maintain support for both x64 and arm64 architectures
- Keep error handling and user feedback intact

## Commit Message and PR Title Format

This repository uses [Conventional Commits](https://www.conventionalcommits.org/). All commit messages and PR titles must follow this format:

```
<type>: <subject>
```

### Types

| Type    | Use For                                           |
|---------|---------------------------------------------------|
| `feat`  | New features or functionality                     |
| `fix`   | Bug fixes                                         |
| `chore` | Build, CI/CD, dependencies, maintenance           |
| `test`  | Adding or updating tests                          |
| `doc`   | Documentation only changes                        |

### Rules

1. **Use lowercase** for type and subject
2. **No period** at the end of the subject
3. **Use imperative mood** ("add" not "added" or "adds")
4. **Keep subject under 72 characters**
5. **Reference PR number** when applicable: `(#123)`
6. **Use the same Conventional Commit format for PR titles**

### Examples

```
feat: add support for branch tracking in add command
fix: handle missing git config gracefully
chore: update dependencies to latest versions
test: add edge case tests for prune command
doc: update readme with new installation method
fix: address edge cases in worktree detection (#17)
chore: add notarization for macos binaries (#15)
```

### Multi-line Commits

For complex changes, add a body separated by a blank line:

```
feat: add self-update command

Allows users to update grove to the latest version or a specific
version directly from the CLI. Supports installing PR preview builds
with the --pr flag.
```

## Code Conventions

### Command Files

Each command in `src/commands/` is implemented as a public function that takes parsed arguments and executes the command logic. Commands are registered in `src/main.rs` using clap's derive macros:

```rust
// In src/main.rs
#[derive(Subcommand)]
enum Commands {
    /// Short description of command
    Example {
        /// Argument description
        name: String,
        /// Flag description
        #[arg(short, long)]
        flag: bool,
    },
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [captainsafia/grove](https://github.com/captainsafia/grove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
