---
trigger: always_on
description: FOP (Filter Orderer and Preener) - Rust CLI tool for sorting and cleaning ad-blocking filter lists. Multi-platform build pipeline publishes to npm with 12+ binary targets.
---

# FOP-RS Project Guide

## Overview
FOP (Filter Orderer and Preener) - Rust CLI tool for sorting and cleaning ad-blocking filter lists. Multi-platform build pipeline publishes to npm with 12+ binary targets.

## Project Structure
```
fop-rs/
├── src/
│   ├── main.rs          # CLI entry, argument parsing, process_location()
│   ├── fop_sort.rs      # Core filter sorting/tidying logic
│   ├── fop_git.rs       # Git integration (commits, PRs, diffs)
│   ├── fop_typos.rs     # Typo detection and fixing
│   ├── fop_checksum.rs  # ABP checksum calculation and verification
│   ├── fop_datestamp.rs # Timestamp and version header management
│   └── tests.rs         # Unit tests for all modules
├── Cargo.toml           # Rust dependencies, version
├── package.json         # npm package config, version
├── install.js           # npm postinstall binary downloader, version
└── .github/workflows/
    └── npm-publish.yml  # Multi-platform build & release
```

## Build Commands
```bash
cargo build --release          # Build optimized binary
cargo clippy                   # Lint check (fix all warnings)
cargo test                     # Run all tests
cargo check                    # Quick syntax check
```

## Version Management
Three files must stay in sync:
- `Cargo.toml`: `version = "X.Y.Z"`
- `package.json`: `"version": "X.Y.Z"`
- `install.js`: `const VERSION = 'X.Y.Z'`

The npm-publish workflow handles this automatically via `npm version patch`.

## Code Patterns

### Preferred Style
```rust
// Use ahash instead of std (and Cow for zero-copy returns)
use ahash::AHashSet as HashSet;
use ahash::AHashMap as HashMap;
use std::borrow::Cow;

// Pre-allocate when size known
let mut vec = Vec::with_capacity(items.len());

// LazyLock for static regex
static PATTERN: LazyLock<Regex> = LazyLock::new(|| Regex::new(r"...").unwrap());

// Inline small hot functions
#[inline]
fn small_function() { }

// Prefer if let for single matches
if let Some(x) = option { } else { }

// Collapsible else-if (clippy will warn)
if a {
} else if b {  // NOT: } else { if b {
}
```

// Use Cow<str> to avoid allocation when returning unchanged input
fn process(input: &str) -> Cow<'_, str> {
    if no_change { Cow::Borrowed(input) } else { Cow::Owned(modified) }
}
```

### Avoid
- `HashMap`/`HashSet` from std (use ahash)
- Unnecessary `.clone()` or `.to_string()` calls (use `Cow<str>` on hot paths)
- `unwrap()` on user input (use proper error handling)
- Nested `else { if }` blocks

## Common Tasks

### Adding a CLI argument
1. Add field to `Args` struct (~line 110 in main.rs)
2. Add default in `Args::parse()` (~line 365)
3. Add CLI parsing in match block (~line 370-460)
4. Add config file support if needed (~line 362)
5. Pass through to `process_location()` if needed
6. Update `print_help()` and `print_config()`
7. Update README.md
8. Add tests in tests.rs if applicable

### Adding a config option
1. Add to `Args` struct
2. Add `parse_bool()` or `parse_list()` call in config loading
3. Document in README.md under Configuration File

### Modifying commit validation
- Edit `check_comment()` in fop_git.rs (~line 277)
- `COMMIT_PATTERN` regex is at line 250
- `valid_url()` validates URLs for A:/P: commits
- Add tests in `test_check_comment()` in tests.rs

### Modifying banned domain detection
- `check_banned_domain()` in fop_sort.rs - main check
- `extract_banned_domain()` - parses domain from rule
- CI mode diff scanning in main.rs (~line 1353)

## Commit Message Format
When `no-msg-check = false`:
| Prefix | Requires | Example |
|--------|----------|---------|
| `A:` | URL | `A: https://github.com/easylist/easylist/issues/123` |
| `P:` | URL | `P: https://forums.lanik.us/viewtopic.php?t=1234` |
| `M:` | Text | `M: Update filters` |

## Config File (.fopconfig)
Located in working directory or home directory.

Key options:
```ini
no-commit = false           # Skip git prompts
no-msg-check = false        # Skip commit validation
check-banned-list = file    # Banned domain list path
ignorefiles = .json,.bak    # Files to skip
history = M: Update,M: Sort # Arrow-key commit history
ci = false                  # CI mode (exit codes)
```

## CI Mode (--ci flag)
- Scans git diff for banned domains (not full files)
- Compares against `origin/master` (PR) or `HEAD~1` (push)
- Respects `ignorefiles` from config
- Exits with code 1 if banned domains found
- Used in GitHub Actions for PR/push checks

## Testing

### Running Tests
```bash
cargo test                     # Run all tests
cargo test test_name           # Run specific test
cargo test -- --nocapture      # Show println! output
```

### Test File Structure (src/tests.rs)
Tests are organized by module:
- **Main.rs tests**: `test_valid_url`, `test_tld_only`, `test_check_comment`, `test_localhost_pattern`
- **fop_sort.rs tests**: `test_remove_unnecessary_wildcards`, `test_convert_ubo_options`, `test_filter_tidy`, `test_sort_domains`
- **Element tidy tests**: Attribute selectors, `*:not()`, `*:has()` preservation
- **:has-text() merge tests**: Combining rules with same base selector

### Adding New Tests
```rust
#[test]
fn test_feature_name() {
    // Arrange
    let input = "...";
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanbr/fop-rs](https://github.com/ryanbr/fop-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
