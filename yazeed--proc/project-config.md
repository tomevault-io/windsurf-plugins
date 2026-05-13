---
trigger: always_on
description: Project-specific instructions for Claude Code.
---

# CLAUDE.md

Project-specific instructions for Claude Code.

## Working Style

**Always ask before committing.** Show the diff or summary and wait for explicit approval before running `git commit`.

**Always ask before pushing.** Never run `git push` or `git push --tags` without explicit user approval. This applies to every push — main branch, tags, all of it.

**Always ask before tagging.** Never run `git tag` without explicit user approval. Tags trigger CI releases to crates.io, npm, Homebrew, Scoop, and Docker — they cannot be easily undone.

**Ask before destructive actions.** This includes `git checkout`, `git reset`, file deletions, or anything that can't be easily undone.

**Show work before publishing.** Before running release workflows, publishing to package managers, or pushing tags, summarize what will happen and wait for approval.

**ALWAYS follow the RELEASE.md checklist for every version.** No exceptions — not for patch versions, not for "trivial" fixes. Every release must complete all steps: pre-release checks (fmt, clippy, test, build, update, audit), manual testing, version bumps, documentation review. Run the checklist, show the results, then ask for approval to commit/push/tag.

## Project Overview

proc is a semantic CLI tool for process management written in Rust. It provides a unified interface for process and port operations across macOS, Linux, and Windows.

**Tagline (use exactly):** "Semantic CLI tool for process management. Target by port, PID, name or path."

**Repository:** https://github.com/yazeed/proc

## Architecture

```
src/
├── main.rs           # CLI entry point, clap setup, command routing
├── lib.rs            # Library exports
├── error.rs          # Error types and exit codes
├── commands/         # One file per command
│   ├── mod.rs        # Exports all commands
│   ├── kill.rs       # proc kill
│   ├── stop.rs       # proc stop (supports --signal)
│   ├── on.rs         # proc on (bidirectional port/process lookup)
│   ├── by.rs         # proc by (filter by name)
│   ├── find_in.rs    # proc in (filter by directory, named find_in to avoid Rust keyword)
│   ├── list.rs       # proc list
│   ├── info.rs       # proc info
│   ├── ports.rs      # proc ports
│   ├── tree.rs       # proc tree
│   ├── stuck.rs      # proc stuck
│   ├── unstick.rs    # proc unstick
│   ├── freeze.rs     # proc freeze (SIGSTOP)
│   ├── thaw.rs       # proc thaw (SIGCONT)
│   ├── orphans.rs    # proc orphans
│   ├── why.rs        # proc why (ancestry tracing)
│   ├── free.rs       # proc free (kill + verify port freed)
│   └── wait.rs       # proc wait (block until exit)
├── core/             # Shared logic
│   ├── mod.rs        # Exports
│   ├── filters.rs    # Shared filter utilities (resolve_in_dir)
│   ├── target.rs     # Target parsing (:port, PID, name) and resolution
│   ├── process.rs    # Process operations
│   └── port.rs       # Port operations
└── ui/               # Output formatting
    ├── mod.rs
    ├── format.rs     # Shared formatting (format_duration, truncate_*, colorize_status)
    └── output.rs     # Printer, OutputFormat (Human/Json)
```

## Code Style

### Command Pattern

Each command follows this structure:

```rust
//! `proc <cmd>` - Brief description
//!
//! Examples:
//!   proc cmd target          # What it does
//!   proc cmd :3000           # Port example
//!   proc cmd --flag          # Flag example

use crate::core::{...};
use crate::error::{ProcError, Result};
use crate::ui::{OutputFormat, Printer};
use clap::Args;

/// Brief doc for --help
#[derive(Args, Debug)]
pub struct CmdCommand {
    /// Target description
    pub target: String,

    /// Skip confirmation prompt
    #[arg(long, short = 'y')]
    pub yes: bool,

    /// Output as JSON
    #[arg(long, short = 'j')]
    pub json: bool,

    /// Show verbose output
    #[arg(long, short = 'v')]
    pub verbose: bool,
}

impl CmdCommand {
    pub fn execute(&self) -> Result<()> {
        // 1. Setup printer
        let format = if self.json { OutputFormat::Json } else { OutputFormat::Human };
        let printer = Printer::new(format, self.verbose);

        // 2. Parse and resolve targets
        let targets = parse_targets(&self.target);
        let (processes, not_found) = resolve_targets(&targets);

        // 3. Warn about not-found targets
        for target in &not_found {
            printer.warning(&format!("Target not found: {}", target));
        }

        // 4. Handle dry-run if applicable
        // 5. Confirm destructive actions (unless --yes)
        // 6. Execute operation
        // 7. Print results

        Ok(())
    }
}
```

### Conventions

- **Unified targets**: `:port`, `PID`, `name` work everywhere
- **Multi-target**: Comma-separated, deduplicated by PID
- **Confirmation**: Destructive commands require `--yes` or interactive confirmation
- **Dry-run**: Destructive commands support `--dry-run`
- **JSON output**: All commands support `--json` for scripting
- **Short flags**: `-y` (yes), `-j` (json), `-v` (verbose)

### Shared Utilities

Use the shared modules instead of writing local helpers:

- **`crate::core::resolve_in_dir`** — resolves `--in` directory paths (handles `.`, relative, absolute)
- **`crate::ui::format_duration`** — formats seconds as human-readable duration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yazeed/proc](https://github.com/yazeed/proc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
