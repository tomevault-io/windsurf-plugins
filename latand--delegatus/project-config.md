---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->
# The product is Delegatus

This repository is Delegatus, called Agent Log Viewer before the rename
(`docs/design/rename-delegatus.md`). Text a person or an agent reads names it
Delegatus. In code, and in these notes, "the Viewer" still names the web server
process as distinct from the runtime host, and the MCP server keeps the key
`viewer`, so its tools stay `mcp__viewer__*`.

<!-- BEGIN:worktree-grouping -->
# Worktree → project grouping (canonical — do not re-break)

Agents run tasks inside **worktree checkouts**. Every session that runs from a
worktree MUST group in the sidebar under its **parent repo's** project — never
as its own lookalike project. This is one algorithm, enforced in
`src/lib/scanner/describe.ts` by `projectInfoFromCwd(cwd)`, which resolves the
parent repo by trying these recognizers in order:

The **pure path recognizers** run first — they need nothing on disk, so they
work identically for live and deleted checkouts:

1. `projectInfoFromClaudeTaskCwd` — Claude scratchpad descendants at
   `<tmp>/claude-<uid>/<encoded-cwd>/<session>/scratchpad/…`; dotted worktree
   containers survive in the encoded cwd and recover the parent project.
2. `worktreeFromPath` — Claude worktrees at `<repo>/.claude/worktrees/<name>/…`
3. `worktreeFromNested` — the `git worktree add worktrees/<name>` (and dotted
   `.worktrees/<name>`) convention: the checkout nests inside the repo, so the
   repo is the path prefix before the first `worktrees`/`.worktrees` segment;
   specialized `.claude`/`.codex` containers are left to #2/#4. The
   first container wins, so a worktree-of-a-worktree groups under the outermost repo.
4. `worktreeFromCodexPath` — Codex worktrees at `~/.codex/worktrees/<hash>/<Repo>`

Only then the **disk-dependent** resolvers, as fallbacks:

5. `worktreeFromGitFile` — any linked git worktree, resolved from its `.git`
   **file** (`gitdir:` pointer) — works **only while the checkout exists on
   disk**. Every live resolution here is written to a persistent map (below).
6. `worktreeFromMemory` — replays a `worktreeFromGitFile` resolution recorded
   (to `state/worktree-map.json`) while the checkout was alive. This is the only
   thing that saves an **arbitrary-path** `git worktree add ../sibling` checkout
   (e.g. `~/.agents/tools/live-log-viewer-<branch>`), which has NO recognizable
   path layout, once it is deleted. Consulted only when no path recognizer
   matched and the cwd is gone.

**The invariant that keeps biting:** a worktree's grouping must survive the
checkout being **deleted**. Any mapping that finds the parent repo only by
reading on-disk git metadata (#5) silently fails afterward and the session
fragments into a phantom lookalike project (`-codex-worktrees-<hash>-<Repo>`,
`…-Projects-<Repo>-worktrees-<name>`, `…-<branch>`, …). Recognize each layout by
**path** (#1–#4) wherever the path reveals the repo; fall back to the persisted
resolution (#6) only for arbitrary sibling paths that cannot. Live and dead
checkouts of the same repo must resolve to the **same** project name.

When adding a new agent/worktree layout: prefer a pure path recognizer beside
#1–#4 and wire it into `projectInfoFromCwd`; only reach for the persisted map
when the path genuinely cannot name the repo. Add a "deleted worktree still
groups under its parent repo" case to `describe.test.ts`. Don't rely on the
checkout being present, and don't invent a second naming scheme.

**The same folder can change key over time.** A plain folder is `dir-<path>`, a
repository with no `origin` is `repo-<local path>`, and the same repository
once an origin is added is `repo-<remote>`. Anything recorded before the move
(an orchestrator seat, its tasks and conversations) keeps the old key while
new pipelines get the new one. `src/lib/projects/succession.ts` records that
move once as an alias in the same map `canonicalProject` reads, and only for a
path-derived source verified against the folder. It runs on scan, at seat tick
boot and sweep, and at designation. A remote change is aliased only when the
forge proves a rename: `src/lib/projects/forgeRename.ts` asks GitHub for the
old and the new name, and only the same numeric repository id joins them. The
old remote comes from `state/project-remotes.json`, which each machine fills
with the remote behind every repository key it resolves, because a key's hash
cannot be reversed. A re-pointed origin (a fork, an unrelated repository) is
never aliased, and neither is a remote this machine never recorded, because
every clone shares a remote id.
<!-- END:worktree-grouping -->

<!-- BEGIN:live-state-and-publication -->
# Local hooks mirror the publication gate

Run `git config core.hooksPath .githooks` once per clone (worktrees inherit it
from their parent repo's config). `pre-push` runs the real
`privacy-publication-gate` (sub-second) from the merge base with commit
checking, and warns when the branch is behind `origin/main` — the state in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Latand/delegatus](https://github.com/Latand/delegatus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
