---
trigger: always_on
description: This repository primarily houses the `compound-engineering` coding-agent plugin and the Claude Code marketplace/catalog metadata used to distribute it.
---

# Agent Instructions

This repository primarily houses the `compound-engineering` coding-agent plugin and the Claude Code marketplace/catalog metadata used to distribute it.

It also contains:
- the Bun/TypeScript CLI that converts Claude Code plugins into other agent platform formats
- additional plugins under `plugins/`, such as `coding-tutor`
- shared release and metadata infrastructure for the CLI, marketplace, and plugins

`AGENTS.md` is the canonical repo instruction file. Root `CLAUDE.md` exists only as a compatibility shim for tools and conversions that still look for it.

## Quick Start

```bash
bun install
bun test                  # full test suite
bun run release:validate  # check plugin/marketplace consistency
```

## Working Agreement

- **Branching:** Create a feature branch for any non-trivial change. If already on the correct branch for the task, keep using it; do not create additional branches or worktrees unless explicitly requested.
- **Merge policy:** All changes to `main` go through pull requests. Direct pushes and direct merges are not allowed; branch protection on `main` enforces this by requiring the `test` status check to pass. The direct path bypasses `release:validate`, the test suite, and PR title validation — past direct merges have caused version drift requiring multi-PR recovery (see `docs/solutions/workflow/release-please-version-drift-recovery.md`).
- **Safety:** Do not delete or overwrite user data. Avoid destructive commands.
- **Testing:** Run `bun test` after changes that affect parsing, conversion, or output.
- **Release versioning:** Releases are prepared by release automation, not normal feature PRs. The repo now has multiple release components (`cli`, `compound-engineering`, `coding-tutor`, `marketplace`). GitHub release PRs and GitHub Releases are the canonical release-notes surface for new releases; root `CHANGELOG.md` is only a pointer to that history. Use conventional titles such as `feat:` and `fix:` so release automation can classify change intent, but do not hand-bump release-owned versions or hand-author release notes in routine PRs.
- **Linked versions (cli + compound-engineering):** The `linked-versions` release-please plugin keeps `cli` and `compound-engineering` at the same version. This is intentional -- it simplifies version tracking across the CLI and the plugin it ships. A consequence is that a release with only plugin changes will still bump the CLI version (and vice versa). The CLI changelog may also include commits that `exclude-paths` would normally filter, because `linked-versions` overrides exclusion logic when forcing a synced bump. This is a known upstream release-please limitation, not a misconfiguration. Do not flag linked-version bumps as unnecessary.
- **Output Paths:** Keep OpenCode output at `opencode.json` and `.opencode/{agents,skills,plugins}`. For OpenCode, command go to `~/.config/opencode/commands/<name>.md`; `opencode.json` is deep-merged (never overwritten wholesale).
- **Scratch Space:** Default to OS temp. Use `.context/` only when explicitly justified by the rules below.
  - **Default: OS temp** — covers most scratch, including per-run throwaway AND cross-invocation reusable, regardless of whether a repo is present or whether other skills may read the files. A stable OS-temp prefix handles cross-skill and cross-invocation coordination equally well as an in-repo path; repo-adjacency is rarely the relevant property.
    - **Per-run throwaway**: `mktemp -d -t <prefix>-XXXXXX` (OS handles cleanup). Use for files consumed once and discarded — captured screenshots, stitched GIFs, intermediate build outputs, recordings, delegation prompts/results, single-run checkpoints. The resulting path is opaque (on macOS it resolves under `$TMPDIR`/`/var/folders/...`) — that is appropriate for throwaway files users are not meant to access.
    - **Cross-invocation reusable**: stable path `/tmp/compound-engineering/<skill-name>/<run-id>/` — **not** `mktemp -d` — so later invocations of the same skill can discover sibling run-ids. Use `/tmp` directly rather than `$TMPDIR` so paths stay accessible: `$TMPDIR` on macOS resolves to `/var/folders/64/.../T/`, which is hostile for users who want to inspect checkpoints, grep them, or copy them out. The per-user isolation `$TMPDIR` provides is not valuable for cross-invocation reusable scratch where users are the intended audience. Use for caches keyed by session, checkpoints meant to survive context compaction within a loose session, or any state where later runs of the same skill need to locate prior outputs.
  - **Exception: `.context/`** — use only when the artifact is genuinely bound to the CWD repo AND meets at least one of:
    - (a) **User-curated**: the user is expected to inspect, manipulate, or manually curate the artifact outside the skill (e.g., a per-repo TODO database, a per-spec optimization log that survives across sessions on the same checkout).
    - (b) **Repo+branch-inseparable**: the artifact's meaning is inseparable from this specific repo or branch (e.g., branch-specific resume state that a user expects to pick up again in the same checkout).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
