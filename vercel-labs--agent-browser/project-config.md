---
trigger: always_on
description: Instructions for AI coding agents working with this codebase.
---

# AGENTS.md

Instructions for AI coding agents working with this codebase.

## Package Manager

This project uses **pnpm**. Always use `pnpm` instead of `npm` or `yarn` for installing dependencies, running scripts, etc. (e.g., `pnpm install`, `pnpm run build`).

## Code Style

- Do not use emojis in code, output, or documentation. Unicode symbols (✓, ✗, →, ⚠) are acceptable.
- In documentation and markdown, never use double hyphens (`--`) as a dash. Use an emdash (—) sparingly when needed. Prefer rewriting the sentence to avoid dashes entirely.
- CLI colored output uses `cli/src/color.rs`. This module respects the `NO_COLOR` environment variable. Never use hardcoded ANSI color codes.
- CLI flags must always use kebab-case (e.g., `--auto-connect`, `--allow-file-access`). Never use camelCase for flags (e.g., `--autoConnect` is wrong).

## Documentation

When adding or changing user-facing features (new flags, commands, behaviors, environment variables, etc.), update **all** of the following:

1. `cli/src/output.rs` — `--help` output (flags list, examples, environment variables)
2. `README.md` — Options table, relevant feature sections, examples
3. `skill-data/core/SKILL.md` (and its `references/`) — so AI agents know about the feature when they load the core skill. Edit `skill-data/core/SKILL.md` for overview/workflow changes; edit `skill-data/core/references/*.md` for detailed reference content. Do **not** put feature content in `skills/agent-browser/SKILL.md` — that file is an intentionally thin discovery stub for `npx skills add` and exists only to redirect agents to `agent-browser skills get core`.
4. `docs/src/app/` — the Next.js docs site (MDX pages)
5. Inline doc comments in the relevant source files

This applies to changes that either human users or AI agents would need to know about. Do not skip any of these locations.

In the `docs/src/app/` MDX files, always use HTML `<table>` syntax for tables (not markdown pipe tables). This matches the existing convention across the docs site.

## Dashboard (packages/dashboard)

- Never use native browser dialogs (`alert`, `confirm`, `prompt`). Use shadcn/ui components (`Dialog`, `AlertDialog`, etc.) instead.
- Use param-case (kebab-case) for all file and folder names (e.g., `session-tree.tsx`, not `SessionTree.tsx`). The `ui/` directory follows shadcn conventions which already uses param-case.

## Releasing

Releases are manual, single-PR affairs. There is no changesets automation. The maintainer controls the changelog voice and format.

To prepare a release:

1. Create a branch (e.g. `prepare-v0.24.0`)
2. Bump `version` in `package.json`
3. Run `pnpm version:sync` to update `cli/Cargo.toml`, `cli/Cargo.lock`, and `packages/dashboard/package.json`
4. Write the changelog entry in `CHANGELOG.md` at the top, under a new `## <version>` heading, wrapped in `<!-- release:start -->` and `<!-- release:end -->` markers. Remove the `<!-- release:start -->` and `<!-- release:end -->` markers from the previous release entry so only the new release has markers.
5. Add a matching entry to `docs/src/app/changelog/page.mdx` at the top (below the `# Changelog` heading)
6. Open a PR and merge to `main`

When the PR merges, CI compares `package.json` version to what's on npm. If it differs, it builds all 7 platform binaries, publishes to npm, and creates the GitHub release automatically. The GitHub release body is extracted from the content between the `<!-- release:start -->` and `<!-- release:end -->` markers in `CHANGELOG.md`.

### Writing the changelog

Review the git log since the last release and write the entry in `CHANGELOG.md`. Follow the existing format and voice. Group changes under `### New Features`, `### Bug Fixes`, `### Improvements`, etc. Bold the feature/fix name, then describe it concisely. Reference PR numbers in parentheses.

Wrap the release notes (everything between the `## <version>` heading and the previous version) in markers so CI can extract them for the GitHub release. Only the current release should have markers; remove the `<!-- release:start -->` and `<!-- release:end -->` markers from any previous release entry:

```markdown
## 0.24.1

<!-- release:start -->
### Bug Fixes

- Fixed **baz** not working when qux is enabled (#1235)

### Contributors

- @ctate
<!-- release:end -->

## 0.24.0

### New Features

- **Foo command** - Added `foo` command for bar (#1234)
```

Include a `### Contributors` section listing the GitHub usernames (with `@` prefix) of everyone who contributed to the release. Check the git log between the previous tag and HEAD to find them.

Do not prefix entries with commit hashes. Do not use the changesets `### Patch Changes` / `### Minor Changes` headings. Use descriptive section names instead.

### Docs changelog

The docs changelog at `docs/src/app/changelog/page.mdx` mirrors `CHANGELOG.md` but uses a slightly different format. Each entry uses:

- A `v` prefix on the version (e.g. `## v0.24.0`)
- A date line with the full date: `<p className="text-[#888] text-sm">March 30, 2026</p>`
- A `---` separator between entries

Match the existing style in that file.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
