---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the source code for the eunomia-bpf project website (https://eunomia.dev). The site provides comprehensive tutorials and documentation for eBPF programming, the eunomia-bpf framework, bpftime, and related projects.

This repository is also responsible for the complete operation of the
eunomia-bpf community, including documentation/site operations, community
coordination, media publishing, and external communications/promotion.

Public media work separates voice from ownership. Personal accounts carry the
maintainer's judgment, research choices, and engineering experience;
organization and project accounts carry formal releases and commitments.
Eunomia.dev is the institutional canonical archive and portfolio, with
eunomia-bpf, AgentSight, ActPlane, bpftime, papers, tutorials, and talks as the
core public evidence assets. Optimize platform posts for native account trust,
discussion, and community reach rather than website ranking alone.

## Required Workflow

For open-source code, documentation, synchronization, CI, release-readiness, or
PR-bound changes in this repository, use the `oss-change-workflow` skill before
editing. Follow its scope-control, validation, review, and CI guidance.

Before adding or keeping planning material in the repository, classify it by
lifespan. Short-term fixes, cleanup backlogs, one-off audits, and tactical
remediation plans should be tracked as GitHub issues, not long-lived draft
documents. Long-term decisions that are reusable, public-safe, and durable may
remain as documentation or skill guidance. When a document mixes both, split it:
move stable strategy or workflow guidance into the appropriate durable doc/skill,
open issues for concrete short-term fixes, then remove the temporary planning
file.

After every external platform publishing session, do a short publishing
lessons pass before reporting completion. Any concrete problem encountered
during drafting, preview, publishing, or public-page QA must be recorded in the
matching publisher skill or reference file, so the same platform mistake does
not recur in the next launch.

For long-form publishing on any external platform, prepare the platform-specific
upload/import artifact locally before opening the editor whenever practical.
Use a temporary file or `draft/media/YYYY-MM-DD/<source-slug>/<platform>.md`
for the final title, body/H1 shape, image URLs or upload assets, table/code
fallbacks, links, tags/categories, and source/project note. Use platform
editors for import/upload, metadata/settings, preview, and QA, not for large
rewrites or fragile structural repairs. No platform requires a visible
canonical/source link in the article body; include one only when it helps the
reader.

If a same-day content-operations log is genuinely useful, write only
`draft/media/YYYY-MM-DD/run-log.md`. Do not create monthly daily-log files or
standalone figure inventories, platform-hook notes, per-article publish-QA
notes, or other disposable files whose only purpose is to prove that a workflow
step happened.

Work in this checkout directly on `main`. Do not create or switch to another
branch or worktree, and do not open a pull request. Before committing, inspect
the worktree, stage only explicit intended paths, run the smallest relevant
validation, and preserve unrelated user changes. Commit the validated change on
`main`, rebase onto `origin/main` when the remote has advanced, and push `main`
directly. This standing repository rule applies to daily `draft/` and
`.agents/skills/` maintenance as well as code, site, build, and documentation
work unless the user explicitly replaces it.

### Branch And Parallel Work

Treat this checkout as shared with the user and possibly other agents. Before
any branch, worktree, stash, rebase, reset, or commit operation, run
`git status --short --branch` and understand the current branch, upstream state,
dirty tracked files, and untracked files.

Keep this checkout on `main`; branch changes are prohibited by the standing
repository workflow. Preserve user work in place unless the user explicitly
asks you to move, stash, or discard it.

When another person or agent may be working in parallel, keep the scope narrow,
edit only requested files, and stage with explicit pathspecs. Never stage
unrelated dirty files or broad untracked directories.

Do not set a timeout when invoking a Subagent. Let it finish naturally; stop it
only when the user explicitly cancels it, its task has become obsolete, or it
has been confirmed stuck.

### Tech stack (current)

The site is rendered by a **custom Next.js + React + Tailwind CSS frontend** living in `app/`, statically exported to plain HTML/CSS/JS. It is **not** a runtime MkDocs site anymore.

- **Next.js** (pages router under `app/pages/`, `output: "export"` static export) — note the directory is named `app/` but uses the *pages* router, not the App Router
- **React 19** + **TypeScript** for components in `app/components/` and content/loader logic in `app/lib/`
- **Tailwind CSS** (`app/tailwind.config.ts`) for styling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eunomia-bpf/eunomia.dev](https://github.com/eunomia-bpf/eunomia.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
