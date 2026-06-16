---
trigger: always_on
description: pinprick is a CLI tool for GitHub Actions supply chain security. It pins action references to full SHAs, checks for updates, and audits pinned actions for runtime fetch patterns that bypass pinning (e.g., `curl ... latest`).
---

# pinprick — Claude Project Context

pinprick is a CLI tool for GitHub Actions supply chain security. It pins action references to full SHAs, checks for updates, and audits pinned actions for runtime fetch patterns that bypass pinning (e.g., `curl ... latest`).

## Project overview

- **Language:** Rust (2024 edition)
- **Platform:** macOS, Linux
- **Architecture:** Single binary CLI with six subcommands (`audit`, `clean`, `completions`, `pin`, `score`, `update`)
- **License:** AGPL-3.0-only
- **Dependencies:** clap/clap_complete (CLI), tokio (async), reqwest (HTTP), serde/serde_norway (parsing), regex (pattern matching), colored (terminal output), toml (config parsing)

## Repository structure

```
pinprick/
├── Cargo.toml
├── build.rs                  # Embeds audited-actions/ into binary at compile time
├── src/
│   ├── main.rs              # Entry point, clap CLI definition, command dispatch
│   ├── audit.rs             # Audit command: scan workflows + action source for runtime fetches
│   ├── audit_patterns.rs    # Compiled regex patterns for shell/JS/Docker fetch detection
│   ├── audited_actions.rs   # Layered lookup: bundled → local cache → remote → GitHub API
│   ├── auth.rs              # GitHub token resolution (GITHUB_TOKEN env → gh auth token fallback)
│   ├── config.rs            # TOML config file loading (.pinprick.toml, ~/.config/pinprick/)
│   ├── github.rs            # GitHub API client (tag→SHA, releases, file trees)
│   ├── output.rs            # Human-readable (colored) and --json output formatting
│   ├── pin.rs               # Pin command: resolve tags to SHAs, rewrite files
│   ├── score.rs             # Score command: compute a posture grade per docs/scoring.md
│   ├── update.rs            # Update command: check pinned actions for newer releases
│   └── workflow.rs           # Regex-based uses: line scanning, ActionRef types
├── audited-actions/          # Pre-audited action SHAs (bundled into binary)
├── docs/                     # Specs (scoring rubric, etc.) — source of truth for behaviors
├── scripts/                  # Helper scripts (release notes formatting)
├── site/                     # Astro Starlight docs site (pinprick.rs)
├── justfile                  # Task runner (build, test, lint, check)
├── rustfmt.toml              # Rustfmt configuration (2024 style edition)
├── .github/
│   ├── workflows/           # CI, CodeQL, zizmor, release, deploy-site, pinprick-audit, audit-actions
│   ├── dependabot.yml       # Dependabot for GitHub Actions, Cargo, and npm
│   └── FUNDING.yml
└── .gitignore
```

## Architecture

### Commands

- `pinprick pin [PATH] [--write]` — Scan `.github/workflows/*.yml`, resolve action tag refs to full SHAs via GitHub API. Dry-run by default (exits 1 when there are unpinned actions). `--write` rewrites files with `@sha # tag` format. Skips already-pinned (SHA) refs. Warns on branch refs (`@main`) and sliding tags (`@v4`), resolving sliding tags to exact versions.
- `pinprick update [PATH] [--write] [--only PATTERN]` — Check SHA-pinned actions for newer releases. Dry-run by default, `--write` to apply changes. `--only` restricts the check to actions whose `owner/repo` contains the given substring.
- `pinprick audit [PATH] [--verbose] [--sarif]` — Scan for runtime fetch patterns that bypass pinning. Without a GitHub token, scans only local `run:` blocks. With a token, also fetches and scans action source code (JS/TS, Python, Dockerfiles, action.yml). `--verbose` shows allowed matches. `--sarif` outputs SARIF 2.1.0 for GitHub code scanning.
- `pinprick score [PATH] [--html]` — Compute a supply-chain posture score (0–100, letter grade A–F) for a repository's workflows. Implements the public rubric in `docs/scoring.md` (rubric v0.6.0). The offline rules (`pin.*`, `workflow.*`, `source.unverified`, `runtime.*`) need no token; with a token it additionally emits the token-gated `source.archived` and `source.advisory` rules. `runtime.*` rules reuse the `audit` shell pipeline against each workflow's `run:` blocks, distinguishing pipe-to-shell (-20) from severity-graded fetches (-15/-8/-3). `source.unverified` is an informational zero-point note — publisher outside the trusted baseline (`actions`, `github`) and the `trusted-owners` list in `.pinprick.toml`; it never affects the score or the gate. Exits 1 when any finding deducts points — zero-point informational notes never gate (matches `audit` for CI gating); outputs JSON with `--json` or a self-contained HTML report with `--html` (mutually exclusive with `--json`).
- `pinprick clean` — Remove locally cached audit results (`~/.cache/pinprick/audited/`).
- `pinprick completions <SHELL>` — Generate shell completions for bash, zsh, fish, etc.

### Global flags

- `--json` — Output as JSON for CI integration
- `--color auto|always|never` — Control color output
- `--version` / `-V` — Print version

### YAML handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starhaven-io/pinprick](https://github.com/starhaven-io/pinprick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
