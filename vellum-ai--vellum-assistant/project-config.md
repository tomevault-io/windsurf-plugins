---
trigger: always_on
description: Bun + TypeScript monorepo with multiple packages:
---

# Vellum Assistant — Agent Instructions

## Project Structure

Bun + TypeScript monorepo with multiple packages:

- `assistant/` — Main backend service (Bun + TypeScript)
- `cli/` — Multi-assistant management CLI (Bun + TypeScript). See `cli/AGENTS.md`.
- `clients/` — Client apps (macOS/iOS/etc). See `clients/AGENTS.md` and platform docs like `clients/macos/AGENTS.md`.
- `gateway/` — Channel ingress gateway (Bun + TypeScript)
- `packages/` — Shared internal packages (e.g. `ces-contracts` for CES wire-protocol schemas)
- `scripts/` — Utility scripts
- `skills/` — First-party skill catalog (portable skill packages). See `skills/AGENTS.md` for contribution rules and portability requirements.
- `.claude/` — Claude Code slash commands and helper scripts (see `.claude/README.md`). Most commands are shared from [`claude-skills`](https://github.com/vellum-ai/claude-skills) via symlinks; repo-local commands (`/update`, `/release`) live in `.claude/skills/<name>/` as local skill directories. The `/update` command uses `vellum ps`, `vellum sleep`, and `vellum wake` to manage assistant lifecycle.

## Intellectual Honesty

Defend your technical positions. If you change your mind, explain what new information changed it — not just that the user questioned it. Do not flip-flop to agree with the user; sycophantic responses erode trust and lead to worse outcomes.

When making recommendations, consider multiple angles — trade-offs, failure modes, alternative approaches — and arrive at a strong, evidence-backed conclusion before presenting it. Vague or hedged suggestions waste time; a clear recommendation with explicit reasoning is always more useful, even if the user ultimately disagrees.

## Development

- **Bun PATH**: Run `export PATH="$HOME/.bun/bin:$PATH"` before any bun/bunx commands.
- **Imports**: All imports use `.js` extensions (NodeNext module resolution).
- **Package manager**: Use `bun install` for dependencies (each package has its own `bun.lock`).

```bash
cd assistant && bun install          # Install dependencies
cd assistant && bunx tsc --noEmit    # Type-check
cd assistant && bun test src/path/to/changed.test.ts  # Run tests
cd assistant && bun run lint         # Lint
```

## Dependencies

This project is licensed under MIT. All dependencies must have MIT-compatible licenses (MIT, Apache-2.0, ISC, BSD-2-Clause, BSD-3-Clause, Unlicense, or similar permissive licenses). Do not add dependencies with copyleft licenses (GPL, AGPL, LGPL, SSPL, EUPL) or proprietary/restrictive licenses without explicit approval.

**Version pinning**: Always use exact versions in `package.json` — no `^` or `~` prefixes. Use `bun add --exact` (or `bun add -E`) when adding packages. The root `bunfig.toml` sets `[install] exact = true` to enforce this by default (bun walks parent directories, so it applies to all packages). See [Bun docs on `--exact`](https://bun.sh/docs/cli/add#exact).

When adding a new dependency:
1. Check its license in the package's `package.json` or LICENSE file.
2. Dual-licensed packages (e.g. "MIT OR GPL-3.0") are acceptable — we use them under the MIT-compatible option.
3. If unsure about compatibility, flag it in the PR for review.
4. Verify the version in `package.json` is pinned to an exact version (no `^` or `~`).

### GitHub Actions

All `uses:` steps in `.github/workflows/**` and `.github/actions/**` must pin to a 40-character commit SHA with a trailing `# vX.Y.Z` comment (e.g. `actions/checkout@a1b2c3... # v6.0.2`). Never use a bare major tag (`@v6`) or a floating version tag (`@v6.0.2`) on its own — SHAs are immutable while tags can be force-moved, so SHA pinning is the GitHub security-hardening recommendation. To upgrade: look up the new tag's commit SHA with `gh api repos/<owner>/<repo>/commits/<tag> --jq .sha`, then replace both the SHA and the trailing comment. For actions that don't publish `vX.Y.Z` tags (e.g. `dawidd6/action-download-artifact`, which tags only bare majors), pin to the SHA with a `# vN` trailing comment instead.

### Swift SPM

In `clients/Package.swift` and any future `Package.swift`, use `.package(url: ..., exact: "X.Y.Z")`. Do not use `.package(url: ..., from: "X.Y.Z")` or other range syntax — the `from:` form silently pulls in new minor/patch releases on each `swift package resolve`.

### Docker base images

In every `Dockerfile`, `FROM` lines must pin the base image to both an exact version tag and an `@sha256:` digest (e.g. `FROM debian:trixie-slim@sha256:...`). Rebuild the digest reference when intentionally upgrading. Do NOT pin `apt-get install` package versions inside Dockerfiles — Debian rotates them out of APT quickly; rely on the base-image digest for reproducibility instead.

### Tool versions

Bun and Node are tracked as separate toolchains; each has its own set of files that must stay in sync. When bumping any file in a set, bump all of them in the same PR so the repo never has drifted copies.

- **Bun**: `.tool-versions`, `setup.sh`, all `bun-version:` workflow inputs, and all production `Dockerfile` bun installs must reference the same exact version string.
- **Node**: `.nvmrc` and every workflow `node-version:` input must reference the same exact version string. (`.nvmrc` is Node-only and is intentionally not tied to the Bun version.)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vellum-ai/vellum-assistant](https://github.com/vellum-ai/vellum-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
