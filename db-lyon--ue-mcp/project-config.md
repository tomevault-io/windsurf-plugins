---
trigger: always_on
description: Operating guide for Claude Code (and any AI agent) working in this repo. Shared team knowledge — checked into the repo so every contributor gets the same rulebook.
---

# CLAUDE.md

Operating guide for Claude Code (and any AI agent) working in this repo. Shared team knowledge — checked into the repo so every contributor gets the same rulebook.

## Repo at a glance

- **TS server** (`src/`) — the MCP server. Wraps the UE bridge over WebSocket, exposes 19 category tools with 440+ actions.
- **C++ plugin** (`plugin/ue_mcp_bridge/`) — the editor-side bridge. Lives in `Private/Handlers/*.cpp`, registers actions with `FMCPHandlerRegistry`.
- **Test project** (`tests/ue_mcp/`) — the dedicated UE project used for smoke testing. The plugin is deployed here from `plugin/` via the deployer. This is the **only** safe target for live tests.
- **Docs** (`docs/`) — MkDocs site. `docs/release-notes-X.Y.Z.md` is the canonical release body.

Edit only under `plugin/ue_mcp_bridge/`. The deployer syncs to `tests/ue_mcp/Plugins/UE_MCP_Bridge/` — never hand-copy.

## Development workflow

### Git and commits

- **Atomic commits per logical change.** One fix or feature per commit with a clear message explaining the *why*. Don't batch unrelated changes.
- **No batching "housekeeping" commits** that sweep up everything at once. If you wrote five fixes, write five commits.
- **Claude owns git.** Stage, commit, push when ready. Don't push the user into running git commands.

### Versioning

**Hard rule: patch-level bumps only.** Version bumps in this repo are always `X.Y.Z → X.Y.(Z+1)`. Never increment major or minor without explicit sign-off. This holds even for genuinely large features — the answer is still a patch bump, and if you think otherwise, ask first.

- Bump `package.json` version.
- Commit and push. **Do not create git tags.** CI detects the version bump on `main` and publishes to npm + creates the GitHub release automatically.

### Building the plugin

- TS: `npx tsc --noEmit` for type-checking, `npx tsc` for emit. Build output goes to `dist/`.
- UE C++: `npm run build`. This calls `scripts/build.js` which invokes Unreal's Build.bat against `tests/ue_mcp/ue_mcp.uproject`.
- `npm run up:build` stops the editor, builds, and relaunches. Use when iterating live.
- The deployer (`scripts/deploy.mjs`, also called implicitly by `npm run up`) syncs `plugin/` → `tests/ue_mcp/Plugins/UE_MCP_Bridge/`. Run it after plugin source edits before building.

### Smoke tests — REQUIRED

`npm run test:smoke` exercises every registered handler via the live WebSocket bridge. **Run this before shipping a release.**

- Target **only** `tests/ue_mcp/ue_mcp.uproject`. Confirm the MCP connection via `project(get_status)` before running. If the editor is connected to anything else (the user's real project, another workspace), abort.
- Smoke tests execute real mutations (create blueprints, delete assets, modify levels). A misrouted run against a real project can corrupt an active editor session.
- 440+ handlers. Pass = every handler responds either with success or an expected parameter-validation error. Any timeout or `Unknown method` is a real failure.

### Clean plugin rebuild recipe

If new handlers return `"Unknown method"` at runtime even though source + build reported success:

1. Delete `tests/ue_mcp/Plugins/UE_MCP_Bridge/` entirely.
2. Delete any `*.patch_*.{dll,pdb,lib,exp}` files under `tests/ue_mcp/Binaries/Win64/`. Live Coding will otherwise load stale patches on top of a fresh DLL.
3. Redeploy (`node scripts/deploy.mjs`), then `npm run build`.

UBT's incremental build + Live Coding can mask registration failures from earlier compile errors. A clean rebuild surfaces the real error.

## Release process

CI **gates the publish job** on a single pre-staged input: the draft GitHub release for `vX.Y.Z`. The draft body must begin with YAML frontmatter declaring a `headline:` array. CI parses, validates against the regex enforced by `scripts/release-headline.mjs`, joins with ` · `, posts the `landing/headline` commit status itself, strips the frontmatter from the body, then promotes the draft. No manual `gh api ... statuses` step.

1. **Author release notes locally** with frontmatter. Notes file lives anywhere (gitignored, scratch, /tmp). The frontmatter must be the first thing in the file:
   ```yaml
   ---
   headline:
     - First feature noun phrase
     - Second feature noun phrase
     - Third feature noun phrase
   ---

   ## vX.Y.Z

   One-line summary.

   ### Server / Bug fixes / Internals
   ...
   ```
   **Headline rules** (enforced by CI; format violations fail the publish job):
   - 1-6 items, each 3-30 chars (keep them tight; concrete noun phrases).
   - Allowed characters: letters, digits, spaces, `_ - / ( ) . , +`.
   - **Forbidden:** `:`, `;`, `?`, `!`, the `·` joiner, leading/trailing whitespace.
   - Joined string (`items.join(" · ")`) must be ≤140 chars.
   - Style: concrete noun phrases naming features. **No sentences. No editorializing the value.** Match the historical voice (see prior releases on the GitHub releases page).

2. **Create the draft release:**
   ```bash
   gh release create vX.Y.Z --draft --notes-file /path/to/local-notes.md
   ```

3. **Bump `package.json` version, commit, push.** That's it - CI takes over.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [db-lyon/ue-mcp](https://github.com/db-lyon/ue-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
