---
trigger: always_on
description: mdlint is an **opinionated Markdown formatter** first, linter second — analogous to ruff or gofmt.
---

# mdlint

@README.md

## Project Philosophy

mdlint is an **opinionated Markdown formatter** first, linter second — analogous to ruff or gofmt.
The formatter (`mdlint format`) enforces a canonical style by rewriting files. The linter
(`mdlint check`) reports violations that fall outside what the formatter can fix automatically.

Core principles: correctness over performance, type safety, minimal code, no duplication,
comprehensive testing.

## Architecture

```text
src/
  main.rs / lib.rs       # Entry point and library root
  args.rs                # CLI argument definitions (clap)
  config/                # TOML config loading, types, and hierarchical merging
  glob/                  # File discovery (ignore crate) and glob matching
  markdown/              # pulldown-cmark wrapper with position tracking
  lint/                  # Rule trait, registry, engine, Violation type
    rules/               # Individual rule implementations (md001.rs, etc.)
  fix/                   # Auto-fix framework
  formatter/             # Canonical markdown rewriter (mdlint format)
  format/                # Output formatters (default, JSON, JUnit, SARIF)
  logger/                # Log level handling
  error.rs / types.rs    # Shared types and error definitions
```

## Lessons Learned

### Configuration System

- TOML is the config format (`mdlint.toml` or `.mdlint.toml`); hierarchical discovery walks up from cwd
- Config merging: later (closer to root) configs override earlier; arrays extend rather than replace
- Front matter: string-based detection for YAML (`---`) and TOML (`+++`) delimiters avoids regex overhead

### File Discovery and Globbing

- Use `ignore` crate for gitignore-aware traversal; requires actual git repo to respect `.gitignore`
- Relative path matching: canonicalize root path, use `strip_prefix()` before glob matching
- Exclude pattern normalization: simple names like `node_modules` → `**/node_modules/**`
- Markdown extensions: md, markdown, mdown, mkdn, mkd, mdwn, mdtxt, mdtext

### Markdown Parsing

- pulldown-cmark: return `impl Iterator<Item = Event<'a>>` to avoid lifetime complexity
- Position tracking: cumulative byte offsets with `line.len() + 1` (accounting for newlines)
- Extensions enabled: tables, footnotes, strikethrough, tasklists, heading attributes

### Rule System

- `Rule` trait: `name()`, `description()`, `tags()`, `check(&MarkdownParser, Option<&Value>)`,
  `fixable()` (default false)
- Registry pattern: `HashMap`-based with `create_default_registry()`
- Rules parse their own config from `Option<&Value>`

### Formatter

- Architecture: emit canonical text directly from pulldown-cmark events; no separate IR needed
- State machine tracks previous block type and inserts blank lines before each new block element
- Idempotency is a hard requirement: `format(format(x)) == format(x)`; proptest found real bugs
- Hard line breaks: trailing-space syntax (two spaces + `\n`) must become backslash continuation
  (`\\\n`) before trailing-whitespace stripping, otherwise the line break is lost
- Code blocks inside list items must be indented by the marker width (3 spaces for an ordered
  `1.` marker, 2 for an unordered `-` marker) to remain inside the list; tracked via
  `list_item_widths` stack in `FormatterState`;
  pulldown-cmark strips the indent on parse so the formatter must re-add it on emit
- `src/formatter/mod.rs` = canonical markdown rewriter; `src/format/` = output formatters
  (JSON, SARIF, JUnit, default) — different concerns, different directories
- Raw HTML blocks and code block contents are passed through verbatim

### Code Quality

- All checks run via `prek run -a` (defined in `prek.toml`), managed by `mise` (`mise.toml`)
- `prek run -a` runs hooks sequentially with `fail_fast = true`:
  trailing-whitespace → end-of-file-fixer → actionlint → hadolint → tombi check → tombi format →
  clippy (with `--fix`) → rustfmt → cargo test → mdlint check (dogfood) → mdlint format (dogfood)
- With `fail_fast`, each failure is one thing to fix; re-run after fixing until all hooks pass
- Formatting hooks (rustfmt, tombi format, end-of-file-fixer, trailing-whitespace) modify files in
  place and report failure — re-running immediately after often passes with no further changes needed
- Clippy runs as errors: `-D warnings`; autofix (`--fix --allow-dirty`) applied before tests
- Common fixes: `unwrap_or()` over manual `is_some()`, iterators over range loops, `!is_empty()`
- Prefer functional patterns: `str::replace` over char loops, `"x".repeat(n)` over push loops,
  bitflags OR operator over `.insert()` chains, `level as u8` over manual match-to-discriminant
- Extract long inline `match` arms to named methods; keep match arms as single expressions
- Avoid bare `{}` scope blocks to limit variable lifetimes — restructure with `if let` or extract
- Keep comments for *why*, not *what*; delete comments that restate adjacent code

### Testing Strategy

- Test business logic, not libraries — focus on merge algorithms, discovery patterns, rule logic
- `cargo test --lib` for unit tests via `src/lib.rs`; `tests/compatibility.rs` for Docker-based tests
- Compatibility tests skip gracefully when Docker is unavailable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swanysimon/mdlint](https://github.com/swanysimon/mdlint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
