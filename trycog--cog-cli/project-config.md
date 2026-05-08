---
trigger: always_on
description: - *NEVER* attempt to use any historically destructive git commands
---

# Rules

- *NEVER* attempt to use any historically destructive git commands
- *ALWAYS* make small and frequent commits
- This project is using Zig 0.15 and you must respect the avalable API for that
- All tests must pass, do not ignore failing tests that you believe are unreleated to your work. Only fix those failing tests after you've completed and validated your work. The last step of any job you do should be to ensure all tests pass.
- *NEVER* attempt to launch the Zig documentation, it is a web app that you cannot access. Instead you *MUST* search the documentation on the ziglang website
- All new features must include `debug_log.log()` calls at key decision points and IO boundaries (network calls, file operations, subprocess spawns, config resolution). These are no-ops when `--debug` is not active.
- When improving Cog support for any one agent integration, proactively review all other supported agents and implement the same or the closest equivalent improvement everywhere their host capabilities allow. If an improvement cannot be replicated, document the host limitation and keep the support matrix aligned.

## Release Process

When the user says "release" (or similar), follow this procedure:

### 1. Determine the version

- If the user specifies a version, use it.
- *MUST* read the current version from `build.zig.zon` first and treat that source-code version as the canonical baseline for the next release. Do not derive the baseline version from git tags, commit messages, or GitHub releases when they disagree with the source tree.
- Do not bump to a new major version while the project is still on `0.x` unless the user explicitly instructs you to start a `1.x` (or higher) release. When the project is still on `0.x`, default to the appropriate `0.x` bump even if the changes would normally look "major" under full SemVer.
- Otherwise, analyze the unreleased commits since the last release commit/tag that matches the source-code version lineage and apply [Semantic Versioning](https://semver.org/):
  - **patch** (0.0.x): bug fixes, build fixes, documentation, dependency updates
  - **minor** (0.x.0): new features, new commands, non-breaking enhancements
  - **major** (x.0.0): breaking changes to CLI interface, config format, or public API
- If tags or history suggest a higher version than `build.zig.zon`, treat that as drift to be corrected instead of as the next release baseline.

### 2. Update version string

Update the version in the single source of truth:
- `build.zig.zon` — `.version = "X.Y.Z",`

(`build.zig` reads the version from `build.zig.zon` via `@import`)

### 3. Review and update README.md

Ensure the README accurately reflects the current state of the project:
- New commands or features are documented
- Removed or renamed features are cleaned up
- Installation instructions are current
- Examples and usage sections match the actual CLI interface

### 4. Update CHANGELOG.md

Follow [Keep a Changelog](https://keepachangelog.com/):
- Add a new `## [X.Y.Z] - YYYY-MM-DD` section below `## [Unreleased]` (or below the header if no Unreleased section exists)
- Categorize changes under: Added, Changed, Deprecated, Removed, Fixed, Security
- Add a link reference at the bottom: `[X.Y.Z]: https://github.com/trycog/cog-cli/releases/tag/vX.Y.Z`
- Each entry should be a concise, user-facing description (not a commit message)

### 5. Commit, tag, and push

```sh
git add build.zig.zon README.md CHANGELOG.md
git commit -m "Release X.Y.Z"
git tag vX.Y.Z
git push && git push origin vX.Y.Z
```

The GitHub Actions release workflow handles the rest: building binaries, creating the GitHub Release, and updating the Homebrew tap.

## CLI Design Language

The Cog CLI has a distinctive visual identity. All terminal output follows these conventions.

### Brand

- **Logo**: "COG" spelled with Unicode box-drawing characters (┌ ┐ └ ┘ ─ │) — see `tui.header()`
- **Tagline**: "Memory for AI agents" in dim text below the logo
- **Primary color**: Cyan (`\x1B[36m`) — the brand accent, used for the logo, section headers, interactive glyphs, and structural elements

### Visual Hierarchy (3 levels)

1. **Bold** (`\x1B[1m`) — primary content: command names, selected items, key labels

<cog>
# Cog

Code intelligence, persistent memory, and interactive debugging.

**Truth hierarchy:** Current code > User statements > Cog knowledge.

## Code Intelligence

For any request to explore, analyze, understand, map, or explain code, use `cog_code_explore` or `cog_code_query`.
Do NOT use Grep, Glob, or shell search commands like `grep`, `rg`, `find`, or `git grep` for code exploration when the Cog index is available.

- `cog_code_explore` — find symbols by name, return full definition bodies, file TOC, and optional architecture summaries. ALWAYS put all symbols into a single `queries` array — never split across multiple calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trycog/cog-cli](https://github.com/trycog/cog-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
