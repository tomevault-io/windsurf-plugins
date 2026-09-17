---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vincent is a local-first control plane for AI coding-agent workloads: a background
daemon owns state and execution (SQLite + git worktrees + agent CLI subprocesses),
and every client — the Bubble Tea TUI, `vincent` CLI subcommands, curl — is a thin
consumer of its localhost REST+SSE API. One Go binary (`cmd/vincent`) serves all
roles.

## Documentation model

Public documentation is feature-first and rooted at `docs/README.md`:
`docs/features.md` presents the product surface, while `getting-started/`,
`guides/`, `platforms/`, `reference/`, `security-model.md`, and `faq.md` explain
how to use it. It is derived from the source, not from planning records: the
config reference tracks `internal/config`, the CLI page the cobra tree, the API
page `internal/api/server.go`'s route table, the TUI key tables
`internal/tui/bindings.go`, and the block reasons the `Reason*` constants. A
change to any of those is a change to its page. The same rule covers the
pictures: every `docs/assets/tui-*.png` is a capture of the running TUI produced
by `scripts/screenshots.sh` (below), never a drawing of one.

GitHub Pages SEO metadata is centralized in `_data/seo_pages.json` and
`_data/why_articles.json`. Keep those entries in sync when public pages or the
"Why vincent is awesome" collection changes. The collection's social cards are
generated from that article data by `scripts/social-cards.py`; do not hand-edit
the generated PNGs in `docs/assets/social/`.

**Product-name style:** write `vincent` with a lowercase `v` everywhere in
prose, headings, labels, metadata, and skill names. Use an initial capital only
when vincent is the first word of a complete sentence. Preserve the exact case of
case-sensitive code and external identifiers, such as the `$VINCENT` gate-script
variable and the WinGet package ID `lezli01.Vincent`; those are not prose.

Implementation work also uses these maintainer records. They are **not**
optional when a change touches the behavior or decision they describe:

- `docs/spec.md` — the product and implementation spec, and the **only** one.
  Section numbers (§6, §12.2, §13.3 …) are used as identifiers throughout the code
  comments. When a comment says "spec §9.1", go read it before changing that code.
  It is deliberately **not versioned**: it describes the system as it is now, and is
  amended in place with dated notes, in the same PR as the code that makes an
  amendment true. Two specs would make every `spec §9.1` citation ambiguous, which
  is why the original version's copy was extracted here rather than forked.
- `docs/tasks/` — planned and in-flight work, one document per piece of work
  (`001-configurable-branch-names.md`), with its decisions. `docs/tasks/README.md`
  is the index and defines the conventions: status markers
  (`[ ]`/`[~]`/`[x]`/`[!]`), `NNN.n` task IDs, updating the index row in the same
  edit, and never deleting a task. Mention the task ID in the PR ("closes 001.4").
- `docs/history/v0-tasks.md` — the **closed** initial-release ledger (70/70,
  released as `v0.1.0`). Frozen: there is no open task in it to claim. Read it for the design
  decisions it carries, which are binding and are cited from code comments
  ("phase 2 decision", "T1.5/T1.6 decision", "PR L decision").
- `docs/gates/` — the manual walkthroughs behind the scripted acceptance gates,
  and the record of when each was last walked.

`skills/vincent-workflows/SKILL.md` and `skills/vincent-triggers/SKILL.md` are
**runtime text, not just documentation**: `skills/embed.go` embeds both, and
`internal/workflow/builtin.go` splices the first into the built-in
`create-workflow` (task 024) and `update-workflows` (task 037) prompts and the
second into `create-trigger` and `update-triggers` (task 098), at build time.
An edit to either file changes what the daemon tells an agent to do. Each
reaches its prompts escaped for `text/template` and re-indented into a YAML
block scalar, so it may contain anything — but each built-in's standing
corrections to its skill live in that built-in's own header, never in the
skill: what asking costs, destination, missing `references/` for
`create-workflow`; that the deliverable is an edit to existing files, that
asking is denied, missing `references/` for `update-workflows`; what asking
costs, staging then `vincent trigger apply`, missing `references/` for
`create-trigger`; that asking is denied, that the deliverable is a staged
proposal, missing `references/` for `update-triggers`. Both trigger headers
carry the never-arm rule, and `vincent trigger apply` enforces it (task 098
decision 3) — never loosen one without the other.

The built-ins are held to the same bar `update-workflows` holds a project's
workflows to (task 037 decision 5): when a workflow feature lands, re-read all
five sources in the same piece of work, use the feature where it applies, and
add the line for it to `update-workflows`' checklist — that checklist is
version-coupled by design, and a feature missing from it is one the built-in
will never propagate. The trigger pair has its own version-coupled checklist,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lezli01/vincent](https://github.com/lezli01/vincent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
