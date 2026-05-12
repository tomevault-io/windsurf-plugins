---
trigger: always_on
description: Rust MCP server giving LLMs IDE-grade code intelligence — symbol-level navigation, semantic search, git integration. Inspired by [Serena](https://github.com/oraios/serena).
---

# codescout

Rust MCP server giving LLMs IDE-grade code intelligence — symbol-level navigation, semantic search, git integration. Inspired by [Serena](https://github.com/oraios/serena).

You are a proficient Rust developer. You follow all known good/scalable patterns. You are honest and recognize your limits and your mistakes, you own them. If you are not sure, you always ask me for feedback.

## Development Commands

See codescout memory `development-commands` for the full command reference.

**Always run `cargo fmt`, `cargo clippy`, and `cargo test` before completing any task.**

**To test changes via the live MCP server, always run `cargo build --release` first**, then restart the server with `/mcp`. The MCP server runs the release binary — dev builds are not picked up.

## Tool Misbehavior Log — MANDATORY

**`docs/TODO-tool-misbehaviors.md` is a living document. You MUST maintain it.**

- **Before starting any task**, read it to know current tool limitations.
- **While working**, watch for: wrong edits, corrupt output, silent failures, misleading errors from codescout's own MCP tools.
- **When you notice anything unexpected**, add an entry to that file **before continuing** — even a one-liner. Capture: what you did, what you expected, what happened, and a probable cause.
- Do not wait until you finish the task. Log it immediately while context is fresh.

This applies to ALL unexpected tool behavior: `edit_file`, `rename_symbol`, `replace_symbol`, `find_symbol`, `semantic_search`, etc.

## Git Workflow

**This is a public repo.** Do not push incomplete or untested work.

### Branch Strategy

- **`master` is protected.** Only cherry-picked, thoroughly tested commits land here.
- **All experimental work goes on the `experiments` branch** (or a dedicated feature branch). Iterate freely there.
- **Cherry-pick to `master`** only after: all tests pass, clippy clean, manually verified via MCP (`cargo build --release` + `/mcp` restart).
- Never commit directly to `master` for in-progress or exploratory work.

### Documenting Features on `experiments`

When adding a feature commit to `experiments`, you MUST include documentation in the same commit:

1. Create `docs/manual/src/experimental/<feature-name>.md` — written as final user-facing
   docs with a single `> ⚠ Experimental — may change without notice.` callout at the top.
2. Add a line to `docs/manual/src/experimental/index.md` linking to the new page.

**Only features, not bug fixes.** Bug fixes need no experimental doc.

**If a feature is removed from `experiments`** (reverted or abandoned), delete its page and
remove its entry from `index.md` in the same commit.

**The experimental docs stay on `experiments` only.** `master`'s `experimental/index.md`
just points to the `experiments` branch on GitHub — it does not list features directly.
This means no cherry-picking of docs to master; the full pages are visible to anyone
browsing the experiments branch.

### Graduating a Feature (`experiments` → `master`)

When cherry-picking a feature to `master`, use `--no-commit` to bundle the doc graduation
into the same commit:

```bash
git cherry-pick --no-commit <sha>
# then make the four graduation changes:
# 1. Move docs/manual/src/experimental/<feature-name>.md to its target chapter
# 2. Remove the `> ⚠ Experimental` callout from the top of the page
# 3. Add the page to docs/manual/src/SUMMARY.md in the right place
# 4. Remove the feature's entry from docs/manual/src/experimental/index.md
git commit -m "feat(...): <description>"
```

The experimental doc page already exists on `experiments` — step 1 is a `git mv`, not a
rewrite. The ⚠ callout and the `experimental/index.md` entry are the only things to remove.

**Rebase note:** Because the graduation commit on `master` includes additional doc changes,
its patch differs from the original `experiments` commit. Git will **not** auto-skip it
during the subsequent `git rebase master` on `experiments`. After rebasing, drop the
now-superseded original commit manually:

```bash
git checkout experiments
git rebase master          # the original feature commit will NOT be auto-dropped
git rebase -i master       # drop the original feature commit from the list
```

### Release Cycle

Full release checklist — run from `master`, never from `experiments` or feature branches.

```bash
# 1. Bump version in Cargo.toml
#    Edit version = "X.Y.Z" in Cargo.toml

# 2. Build release binary and verify
cargo build --release
cargo test
cargo clippy -- -D warnings

# 3. Commit the version bump
git add Cargo.toml Cargo.lock
git commit -m "chore: bump version to X.Y.Z"

# 4. Tag the release
git tag vX.Y.Z

# 5. Publish to crates.io
CARGO_REGISTRY_TOKEN=$(grep CARGO_REGISTRY_TOKEN .env | cut -d= -f2) cargo publish

# 6. Push commit + tag
git push
git push --tags

# 7. Create GitHub release with release notes
gh release create vX.Y.Z --title "vX.Y.Z" --notes "release notes here"

# 8. Rebase experiments on the new master
git checkout experiments && git rebase master
```

**Notes:**
- Token is stored in `.env` (gitignored): `CARGO_REGISTRY_TOKEN=...`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mareurs/codescout](https://github.com/mareurs/codescout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
