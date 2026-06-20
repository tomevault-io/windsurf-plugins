---
trigger: always_on
description: Persist session knowledge (conventions, gotchas, decisions, learnings) into durable documentation: the project's authoritative agent file (CLAUDE.md or AGENTS.md — whichever the project uses, never both), README.md, docs/standards/, ~/source/standards/ (prescriptive cross-project rules), ~/source/knowledgebase/ (descriptive cross-project facts), and other docs/ as needed. **Trigger only on the explicit `/dump` slash command.** Do not trigger on natural-language phrases — the user wants determini
---


# dump

## Pre-flight inventory

The following listings are injected once at skill load. Refer back to them from later steps instead of re-running these commands. If a path is missing the output will say so — treat that as "not present" and act accordingly.

### Root agent/human docs (auto-injected)

ls -1 README.md AGENTS.md CLAUDE.md:

!`ls -1 README.md AGENTS.md CLAUDE.md 2>&1 || true`

Referenced by: Step 2 routing (which root files exist), Step 3 dispatch (target picker for CLAUDE.md vs AGENTS.md), edge cases (no CLAUDE.md/AGENTS.md/README.md).

### Project docs tree  (auto-injected)

ls -la docs/:

!`ls -la docs/ 2>&1 || true`

ls -la docs/standards/:

!`ls -la docs/standards/ 2>&1 || true`

Referenced by: Step 2 routing (existing `docs/` structure determines whether a nugget gets a new file or joins an existing one), Step 3 dispatch (subagent scopes for `docs/standards/` and other `docs/*`), routing heuristics (architectural decision logs, troubleshooting docs).

### Cross-project standards  (auto-injected)

!`${CLAUDE_SKILL_DIR}/scripts/resolve-listing.sh standards`

**Shallow** listing — top level only. For file-level routing of a nugget, read `CLAUDE.md` in the listed path; it is the index mapping topics to files.

Referenced by: Step 3 dispatch (subagent must read the standards repo's `CLAUDE.md` for file-level routing), routing heuristics (where cross-repo patterns live).

### Cross-project knowledgebase  (auto-injected)

!`${CLAUDE_SKILL_DIR}/scripts/resolve-listing.sh knowledgebase`

**Shallow** listing — top level only. For file-level routing of a nugget, read `CLAUDE.md` in the listed path; it is the index mapping topics to files.

Knowledgebase is the **descriptive** sibling of standards: vendor quirks, tool internals, API taxonomies, discovered failure modes — "this is how X works", not "do it this way". A nugget routes here when it describes how an outside vendor/tool/API behaves; it routes to standards when it prescribes how to write code. Hybrid topics live in both, cross-linked.

Referenced by: Step 2 routing (descriptive vs prescriptive split), Step 3 dispatch (subagent must read the knowledgebase repo's `CLAUDE.md` for file-level routing), routing heuristics.

### Repo mode and ownership (auto-injected)

${CLAUDE_SKILL_DIR}/scripts/check-user-owned.sh:

!`${CLAUDE_SKILL_DIR}/scripts/check-user-owned.sh`

Output format depends on mode:

- `MODE: single` + `OWNERSHIP: APPLICABLE|NOT_APPLICABLE: <reason>` — CWD is a git repo.
- `MODE: multi` + one `REPO: <name> APPLICABLE|NOT_APPLICABLE: <reason>` per child repo — CWD is a parent directory containing child repos.
- `MODE: none` — no git repo at CWD and no child repos found.

All modes except `none` also emit:

- `STANDARDS: <absolute-path>|NOT_FOUND` — resolved location of standards repo (checks `~/source/standards/`, `~/standards/`, `./standards/` in order; first git repo wins).
- `KNOWLEDGEBASE: <absolute-path>|NOT_FOUND` — resolved location of knowledgebase repo (same resolution order).

If `STANDARDS` or `KNOWLEDGEBASE` is `NOT_FOUND`, skip that subagent entirely — do not attempt to discover or create it later.

Referenced by: Step 0 multi-repo detection (determines mode, lists child repos, per-repo ownership, and cross-project repo availability), Step 1 environment detect (gates `~/source/standards/` and `~/source/knowledgebase/` subagent dispatch in single-repo mode), Step 4 report (`Skipped` section cites the reason when `NOT_APPLICABLE` or `NOT_FOUND`).

### Remote URLs (auto-injected)

${CLAUDE_SKILL_DIR}/scripts/get-remotes.sh:

!`${CLAUDE_SKILL_DIR}/scripts/get-remotes.sh`

Output: six lines. Both the raw form (whatever `git remote get-url origin` returned — SSH or HTTPS) and the converted public HTTPS form are pre-computed so subagents never have to convert. Standards and knowledgebase paths are resolved using the same logic as `check-user-owned.sh` (`~/source/<name>`, `~/<name>`, `./<name>` — first git repo wins).

- `LOCAL_REMOTE_RAW: <url|none>` — origin of the local repo (cwd or `$1`), as stored in `.git/config`.
- `LOCAL_REMOTE_HTTPS: <url|none>` — same, converted to `https://` browser form.
- `STANDARDS_REMOTE_RAW: <url|none>` — origin of the resolved standards repo.
- `STANDARDS_REMOTE_HTTPS: <url|none>` — same, converted to `https://` browser form.
- `KNOWLEDGEBASE_REMOTE_RAW: <url|none>` — origin of the resolved knowledgebase repo.
- `KNOWLEDGEBASE_REMOTE_HTTPS: <url|none>` — same, converted to `https://` browser form.

The model must use these injected values directly, **never reconstruct them from filesystem paths, do its own SSH→HTTPS conversion, or guess from context**. Anti-pattern reminder: cross-repo references in any written doc must use the `_HTTPS` value verbatim.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jewzaam/claude-skill-dump](https://github.com/jewzaam/claude-skill-dump) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
