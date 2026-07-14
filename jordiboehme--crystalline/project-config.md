---
trigger: always_on
description: Local-first knowledge management for humans and AI agents. Every Domain has exactly one source of truth: by default Markdown files (Engrams) on disk, or, for a virtual domain, the database itself. The search index is always a disposable derived layer; an MCP server and CLI sit on top.
---

# Crystalline

Local-first knowledge management for humans and AI agents. Every Domain has exactly one source of truth: by default Markdown files (Engrams) on disk, or, for a virtual domain, the database itself. The search index is always a disposable derived layer; an MCP server and CLI sit on top.

## Purpose

Crystalline gives an AI agent the capability to learn and evolve instead of starting from zero in every session. An agent is onboarded via a generated routing prompt, taught information through curated domains and stores its learnings and experiences as engrams while it works - becoming a useful and productive peer over time. All user-facing language (README, skills, MCP tool descriptions, routing prompt) is framed around onboarding, teaching, learning and experience rather than notes or documents.

## Vocabulary

- **Domain** - a registered folder of knowledge with a mandatory MANIFEST.md at its root, used for routing
- **Engram** - one markdown file holding a unit of knowledge, with YAML frontmatter (OKF compatible)
- Address scheme: `crystalline://<domain>/<permalink>`

## Workspace

Cargo workspace, Rust edition 2024, pinned toolchain in rust-toolchain.toml.

- `crates/core` (crystalline-core) - format layer: parser, emitter, schema, verify, prompt. Must never depend on async runtimes, databases or ML crates
- `crates/index` (crystalline-index) - Store trait, embedded database backend, sync engine, search, embeddings
- `crates/service` (crystalline-service) - single-instance daemon, MCP server, control protocol
- `crates/cli` (crystalline) - the single user-facing binary

Dependency direction: core <- index <- service <- cli.

## Commands

- Build: `cargo build --release`
- Test: `cargo test --workspace`
- Lint: `cargo clippy --workspace --all-targets -- -D warnings` and `cargo fmt --all --check`
- Style check: `bash scripts/style-lint.sh`

## Toolchain

Rust is managed by rustup, installed via Homebrew (`brew install rustup`). Homebrew links only `rustup` itself into `/opt/homebrew/bin`; the proxies for `cargo`, `rustc`, `clippy` and `rustfmt` live in `/opt/homebrew/opt/rustup/bin`, which is not on the default PATH. If `cargo` is not found, prepend that directory (`export PATH="/opt/homebrew/opt/rustup/bin:$PATH"`) and the commands above run as written.

rustup enforces the `rust-toolchain.toml` pin: inside this repo every proxy resolves to channel 1.96.1 (including clippy and rustfmt) regardless of the default toolchain, and a missing pinned toolchain is downloaded on first use.

## Local folders (gitignored)

- `plans/` - implementation plans. Read the newest plan before starting work; store any new plan here
- `research/` - background research notes. Store any research produced while working here

## Hard rules

- Never reference other knowledge-management tools by name anywhere in this repo, neither in code nor in docs, comments or commit messages. The local plans in `plans/` explain the specifics
- Commit messages: plain, human style. No AI attribution of any kind - no co-author trailers, no generated-with lines
- AI harnesses may be named in user-facing docs (README, skills) only where Crystalline integration is documented
- No emdashes or en dashes in any file - use a plain '-'
- No Oxford comma in markdown files or in application strings
- Temporal semantics: absent `valid_from` = has always been valid, absent `valid_to` = valid forever. Never write sentinel dates like 9999-12-31
- `status` and `type` frontmatter fields are required and non-empty but free form - recommended value sets are guidance, never enforced globally
- Commit after each completed milestone or task
- Use the latest stable versions of dependencies and standards; verify on crates.io rather than assuming
- docs/deployment.md holds the deployment documentation: every scenario (text plus one mermaid chart per scenario), the container guide, the environment variable reference and read-only serving; the README keeps a Deployment section with a one-line-per-scenario table linking into it. Any change that adds or alters a deployment mode (new serve flag, new image variant, new compose example, new transport) must update docs/deployment.md and the README table in the same change
- Store every implementation plan in plans/ with a dated filename before starting work; store research notes in research/
- Delegate implementation to subagents with the model matched to task complexity: opus for design-heavy or intricate work, sonnet for routine or mechanical work. The orchestrator reviews, gates and commits

## Known upstream workarounds


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordiboehme/crystalline](https://github.com/jordiboehme/crystalline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
