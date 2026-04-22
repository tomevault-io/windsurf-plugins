---
trigger: always_on
description: > **Note:** `CODEX.md` and `CLAUDE.md` are symlinks to `AGENTS.md`. Only edit `AGENTS.md` — both files are always in sync.
---

# Guidelines

> **Note:** `CODEX.md` and `CLAUDE.md` are symlinks to `AGENTS.md`. Only edit `AGENTS.md` — both files are always in sync.

- Use semantic commit format, e.g.

```
feat: <short description>

Detailed description of the changes, that can be multiline, but do not artificially create line breaks, as commit messages are automatically wrapped.
- Use bullet points
- When needed
```

- When making changes that affect user-facing behavior (new commands, new flags, changed defaults, new installation methods, etc.), update `README.md` to reflect those changes.

- Whenever any research is done (including by the subagents), it should be documented as a report in docs/research/YYYY-MM-DD-<description>.md. In case of subagents, the need to be instructed to create the report file and on completion return only the path to the report, so that the main agent can read and analyze it.

## Releasing

To release a new version, tag and push. The release workflow triggers automatically on tag push — do NOT also run `gh workflow run release`, as that causes a duplicate build.

**IMPORTANT: Never create releases locally with `gh release create`.** Always let goreleaser handle release creation through the CI pipeline. After tagging and pushing, wait for the workflow to complete, then use `gh release edit` to update the release notes.

**Version numbering:** Decide the version number autonomously based on semver (patch for fixes, minor for features, major for breaking changes). Do not ask the user to confirm the version.

```
git tag vX.Y.Z
git push origin main --tags
```

### CI verification (required)

After pushing the tag, **watch the GitHub Actions `release` workflow**:

```
gh run list --workflow release --limit 10
gh run watch <RUN_ID>
```

If the workflow fails:

1. Inspect logs:
   ```
   gh run view <RUN_ID> --log-failed
   ```
2. Fix the underlying issue in `main` (code/tests/lint/goreleaser config, etc.).
3. Re-run the release by tagging a new version and pushing tags (preferred), or rerun the failed run only if it was clearly transient:
   - Preferred:
     ```
     git tag vX.Y.Z+1
     git push origin main --tags
     ```
   - Transient-only rerun:
     ```
     gh run rerun <RUN_ID> --failed
     ```

Only proceed to release notes once the `release` workflow is green.

### Release notes

After the release workflow completes, edit the GitHub release with human-written notes (not commit logs). Use `gh release edit` to update the release created by goreleaser.

**Process:**

1. Diff against the previous version to understand what changed:
   ```
   git diff vPREVIOUS..vNEW --stat
   git log vPREVIOUS..vNEW --oneline
   ```
2. Write release notes from the user's perspective — what can they do now that they couldn't before?
3. Publish with `gh release edit vX.Y.Z --title 'vX.Y.Z "Codename" — Short Theme' --notes "..."`.

**Format:**

```markdown
One to three sentence TL;DR of what this release is about and why users should care.

## New: Feature name

Brief explanation of what it does and why it matters.

\```bash
# 1-2 practical examples showing real usage
kanban-md <command> <flags>
\```

## New: Another feature

...

## Changed: Behavior change

Explain what changed and what users need to know. Include migration notes if applicable.

## Upgrading

Any steps users need to take, or "No action needed" with explanation of auto-migration.

**Full diff:** [`vPREVIOUS...vNEW`](https://github.com/antopolskiy/kanban-md/compare/vPREVIOUS...vNEW)
```

**Guidelines:**
- Title format: `vX.Y.Z "Codename" — Short Theme` (e.g. `v0.8.0 "Iron Gate" — Claim Enforcement`). The codename is a 1-2 word evocative nickname that creates a memorable association with the release — something vivid and fun that works as a mnemonic (e.g. "Quiet Storm", "Paper Trail", "Red Line"). It should loosely relate to the theme but doesn't need to be literal. **Codenames must be unique across all releases** — check `gh release list` before picking one to avoid duplicates.
- Start with a short TL;DR paragraph (no heading) summarizing the release for someone skimming
- Use `## New:` for new commands/features, `## Changed:` for behavior changes, `## Fixed:` for bug fixes
- Every feature section should include a code example
- End with `## Upgrading` section and a full diff link
- Write for users, not developers — focus on what they can do, not what files changed

## Backward Compatibility

When modifying `config.yml` schema or task file frontmatter, you must ensure backward compatibility:

### Config design principles

- **Collocate column settings with column definitions.** Any configuration that is per-column (e.g. `show_duration`, `require_claim`, WIP limits) must live inside the status/column entry in the `statuses` list — never as a separate top-level list or map. This keeps related settings together and avoids drift between column names and their config.

### Config changes

1. **Bump `CurrentVersion`** in `internal/config/defaults.go`.
2. **Add a migration function** in `internal/config/migrate.go` that transforms the old format to the new one. Register it in the `migrations` map. The migration must increment `cfg.Version`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antopolskiy/kanban-md](https://github.com/antopolskiy/kanban-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
