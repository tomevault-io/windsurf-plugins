---
trigger: always_on
description: *NEVER* change old migrations that are already in git history.
---

## Development Workflow

*NEVER* change old migrations that are already in git history.
*ALWAYS* run the entire test suite before declaring any work is complete.
*ALWAYS* TDD: write failing tests first, implement minimum code to pass, run `mix test` locally, push only when green.

**No fallbacks.** When refactoring, fully commit to the new approach. Remove old code entirely. If the new approach fails, that's a bug to surface, not hide.
j
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
<cog>
# Cog

Code intelligence, persistent memory, and interactive debugging.

**Truth hierarchy:** Current code > User statements > Cog knowledge.

## Code Intelligence

For any request to explore, analyze, understand, map, or explain code, use `cog_code_explore` or `cog_code_query`.
Do NOT use Grep, Glob, or shell search commands like `grep`, `rg`, `find`, or `git grep` for code exploration when the Cog index is available.

- `cog_code_explore` — find symbols by name, return full definition bodies, file TOC, and optional architecture summaries. ALWAYS put all symbols into a single `queries` array — never split across multiple calls.
- `cog_code_query` — `find` (locate definitions), `refs` (find references), `symbols` (list file symbols), `imports` (module/file dependencies), `contains` (parent-child containment), `calls`/`callers` (approximate call graph), `overview` (symbol/file/repo architecture summary). ALWAYS use the `queries` array to combine multiple queries into one call — never make sequential code_query calls that could be batched.
- Include synonyms with `|`: `banner|header|splash`
- Wildcard symbol patterns: `*init*`, `get*`, `Handle?`

Only fall back to Grep, Glob, or shell search commands when the Cog index is unavailable, incomplete for the target code, or the task is about raw string literals, log messages, or other non-symbol text patterns.

### Batching Rules

Both `cog_code_explore` and `cog_code_query` accept a `queries` array.
Making sequential calls to the same tool when a single batched call would
work is an error. Combine them.

- `cog_code_explore`: put ALL symbols into one `queries` array.
  Do not call `cog_code_explore` twice when both calls could be one.
- `cog_code_query`: put ALL queries into one `queries` array. Each entry
  specifies its own `mode`, `name`, `file`, `kind`, `direction`, `scope`.
  Example: symbols for 3 files = one call with 3 entries, not 3 calls.
- For repository-understanding tasks: one initial `cog_code_explore`
  with `include_architecture=true` and `overview_scope="repo"`, then at
  most one targeted follow-up.
- Before making follow-up calls, check whether the answer is already
  present in prior output.
- Prefer `cog_code_query` over raw file reads for architectural questions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BeaconCMS/beacon](https://github.com/BeaconCMS/beacon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
