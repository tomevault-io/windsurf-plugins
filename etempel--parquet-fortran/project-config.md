---
trigger: always_on
description: This file is forward-looking: it captures durable conventions, gotchas, and guardrails to help
---

# Instructions for Claude

This file is forward-looking: it captures durable conventions, gotchas, and guardrails to help
maintain this library and develop new features going forward. It is not a changelog or session
log — do not add entries describing how or when a specific feature was implemented, what a past
session investigated, or a chronological record of development. Only add generalizable guidance
that will still be correct and actionable for a future task, independent of which session
produced it (git history/commit messages are the right place for "what happened when").

**If checked out inside a larger workspace** (e.g. alongside sibling Fortran projects under a
shared `fortran/` directory), check whether `../../fortran/CLAUDE.md` exists and read it too — it
captures conventions shared across those projects (workflow guardrails, documentation/FORD
conventions, generic Fortran/fpm gotchas, testing & coverage conventions) that apply here as well,
unless this file says otherwise. This repository is also developed and used completely standalone,
so that file won't always exist — treat it as supplementary, not required.

## Contents

This file is a reference, not a start-to-finish read — jump to the note you need. Keep this ToC
in sync when adding, removing, renaming, or reordering a heading (see "Documentation structure"'s
working rules).

- [Workflow & guardrails](#workflow--guardrails)
  - [Only modify files inside this repository](#only-modify-files-inside-this-repository)
  - [Report before implementing on analysis/audit requests](#report-before-implementing-on-analysisaudit-requests)
  - [Only apply low-blast-radius renames/refactors](#only-apply-low-blast-radius-renamesrefactors)
  - [`feature_*.md` planning documents](#feature_md-planning-documents)
  - [NEVER delete a `feature_*.md` document — the maintainer archives them](#never-delete-a-feature_md-document--the-maintainer-archives-them)
  - [The `feature_risks.md` standing-risks register](#the-feature_risksmd-standing-risks-register)
  - [Never splice a file with an unanchored `index()` — use Edit, or assert both ends](#never-splice-a-file-with-an-unanchored-index--use-edit-or-assert-both-ends)
  - [Don't run the GitLab CI pipeline yourself](#dont-run-the-gitlab-ci-pipeline-yourself)
  - [The CI-environment Docker image: ask for it, never build it](#the-ci-environment-docker-image-ask-for-it-never-build-it)
  - [Don't commit or push on the main/default branch yourself](#dont-commit-or-push-on-the-maindefault-branch-yourself)
- [Documentation conventions](#documentation-conventions)
  - [New features require tests and docs](#new-features-require-tests-and-docs)
  - [Documentation structure](#documentation-structure)
  - [CONTRIBUTING.md is project-wide workflow only](#contributingmd-is-project-wide-workflow-only--a-tools-own-detail-goes-in-its-header)
  - [A guide page describes the CURRENT state, never a former one](#a-guide-page-describes-the-current-state-never-a-former-one)
  - [Checking documentation links](#checking-documentation-links)
  - [FORD doc-comment conventions](#ford-doc-comment-conventions)
  - [FORD config gotchas](#ford-config-gotchas)
- [Source code structure & conventions](#source-code-structure--conventions)
  - [One program unit per file; filename == unit name](#one-program-unit-per-file-filename--unit-name)
  - [Some `src/*.f90` files are generated — edit the generator, never the output](#some-srcf90-files-are-generated--edit-the-generator-never-the-output)
  - [Nested submodule tree](#nested-submodule-tree)
  - [Group interface bodies into commented `interface` blocks](#group-interface-bodies-into-commented-interface-blocks)
  - [A module procedure cannot implement its own submodule's spec-declared interface](#a-module-procedure-cannot-implement-its-own-submodules-spec-declared-interface)
  - [A separate module procedure must be IMPLEMENTED before it is CALLED in the same submodule](#a-separate-module-procedure-must-be-implemented-before-it-is-called-in-the-same-submodule)
  - [Naming conventions](#naming-conventions)
  - [`parquet_random` is a LEAF; `parquet_sampling` is where anything more goes](#parquet_random-is-a-leaf-parquet_sampling-is-where-anything-more-goes)
  - [Each `parquet_random` generic reads its OWN word space, and a missed tag is silent](#each-parquet_random-generic-reads-its-own-word-space-and-a-missed-tag-is-silent)
  - [Public numeric arguments: provide both int32 and int64 kinds](#public-numeric-arguments-provide-both-int32-and-int64-kinds)
  - [A new process-global parameter goes in `parquet_settings`](#a-new-process-global-parameter-goes-in-parquet_settings-and-a-design-doc-must-say-so)
  - [Role-A MAMLs live in `table_types/`, not `schemas/`](#role-a-mamls-live-in-table_types-not-schemas)
  - [MAML fixture directory: `schemas/`](#maml-fixture-directory-schemas)
  - [Reading MAML source files: shared helper, line-length limit, CRLF handling](#reading-maml-source-files-shared-helper-line-length-limit-crlf-handling)
  - [Error stop messages: include file/schema context](#error-stop-messages-include-fileschema-context)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [etempel/parquet-fortran](https://github.com/etempel/parquet-fortran) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
