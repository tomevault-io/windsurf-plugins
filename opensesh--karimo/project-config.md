---
trigger: always_on
description: When changes impact target repositories (installed KARIMO projects), you MUST complete ALL steps below. **The release is NOT complete until the GitHub Release exists AND is verified via API.**
---

# KARIMO Source Repository Rules

## Release Protocol (MANDATORY)

When changes impact target repositories (installed KARIMO projects), you MUST complete ALL steps below. **The release is NOT complete until the GitHub Release exists AND is verified via API.**

### Preferred Method: Release Script

Use the release script for atomic, verified releases:

```bash
.karimo/scripts/release.sh 8.3.0        # Full release
.karimo/scripts/release.sh 8.3.0 --dry-run  # Preview only
```

The script handles: version bump, commit, tag, push, release creation, and API verification.

### Manual Release Checklist (if not using script)

```
[ ] 1. VERSION BUMP
    - Update `.karimo/VERSION` with new semver
    - Update `version` field in `.karimo/MANIFEST.json` to match

[ ] 2. CHANGELOG ENTRY
    - Add entry to `CHANGELOG.md` under new version header
    - Format: `## [X.Y.Z] - YYYY-MM-DD`
    - Include: Added, Changed, Fixed, Removed subsections as needed

[ ] 3. DOCUMENTATION UPDATES
    - README.md — Update version badge
    - .karimo/docs/ARCHITECTURE.md — Update version header
    - .karimo/docs/COMMANDS.md — If slash commands changed
    - .karimo/docs/GETTING-STARTED.md — If setup flow changed

[ ] 4. COMMIT ALL CHANGES (BEFORE TAGGING)
    - Commit version bump, changelog, AND documentation
    - Push to origin/main
    - CRITICAL: All changes must be committed BEFORE creating tag/release

[ ] 5. CREATE TAG AND RELEASE (ATOMIC)
    - git tag vX.Y.Z
    - git push origin vX.Y.Z
    - gh release create vX.Y.Z --title "vX.Y.Z" --notes "..."
    - Do these in immediate sequence

[ ] 6. VERIFY VIA API (REQUIRED)
    - curl -s "https://api.github.com/repos/opensesh/KARIMO/releases/latest" | grep tag_name
    - Must show your new version
    - If not, the release is BROKEN
```

### CRITICAL: Never Move Tags After Release

**DO NOT run these commands after creating a release:**
```bash
# DANGEROUS - orphans the release:
git tag -d vX.Y.Z && git tag vX.Y.Z  # Moving tag
git tag -f vX.Y.Z                      # Force-updating tag
```

Moving a tag after release creation orphans the GitHub release (makes it a draft pointing to "untagged-*"). This breaks `/karimo:update` for all users.

**If you need to include more commits after release:**
1. Delete the release: `gh release delete vX.Y.Z --yes`
2. Delete the tag: `git tag -d vX.Y.Z && git push origin :refs/tags/vX.Y.Z`
3. Make your additional commits
4. Re-run the release script or checklist from step 1

### STOP CHECK

Before marking a release task complete, verify ALL of these:

1. **Release exists:** `gh release view vX.Y.Z` returns the release
2. **API shows latest:** `curl -s "https://api.github.com/repos/opensesh/KARIMO/releases/latest" | grep tag_name` shows your version
3. **Tag matches HEAD:** `git rev-parse vX.Y.Z` equals `git rev-parse HEAD`

**If any check fails, the release is broken and must be recreated.**

## What Impacts Target Repositories

Changes to these files affect installed projects:
- `.claude/agents/*` — Agent definitions
- `.claude/commands/*` — Slash commands
- `.claude/skills/*` — Skill definitions
- `.karimo/templates/*` — PRD/task templates
- `.claude/KARIMO_RULES.md` — Agent behavior rules

Changes to these files do NOT affect installed projects:
- `install.sh`, `update.sh` — Installer scripts (source-only)
- `CONTRIBUTING.md` — Contribution guidelines
- `.github/workflows/karimo-test-install.yml` — Source-only CI

## Atomic Commit Workflow (MANDATORY)

**Iron Law: COMMIT AFTER EACH LOGICAL UNIT OF WORK — NOT AT THE END**

Bundling all changes into one commit destroys traceability. Each plan phase, task, or logical unit gets its own commit. This is non-negotiable.

### When to Commit

| Trigger | Action |
|---------|--------|
| Plan phase complete | Commit immediately |
| TodoWrite task marked `completed` | Commit that task's changes |
| Bug fix verified | Commit the fix |
| Refactor complete | Commit separately from features |
| Moving to unrelated work | Commit current work first |

### Workflow Integration

1. **During Plan Execution:**
   - Complete a phase/task
   - Verify it works (tests, build, etc.)
   - Stage and commit with descriptive message
   - Mark TodoWrite item as `completed`
   - Move to next task

2. **At End of Work Session:**
   - Show commit summary to user
   - Format: list of commits made with messages
   - Example:
     ```
     ## Commits Made This Session
     - `abc1234` feat(auth): add logout button component
     - `def5678` feat(auth): implement logout API call
     - `ghi9012` test(auth): add logout flow tests
     ```

### Commit Format

Use Conventional Commits:

```
<type>[optional scope]: <description>

[optional body]

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types:** feat, fix, refactor, style, docs, test, chore, perf

**Rules:**
- Imperative mood: "add feature" not "added feature"
- Keep first line under 72 characters
- ALWAYS include Co-Authored-By footer

### Anti-Patterns (STOP if you catch yourself...)

- Saying "I'll commit everything at the end"
- Asking "do you want me to commit?" after all work is done
- Making one commit with unrelated changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensesh/KARIMO](https://github.com/opensesh/KARIMO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
