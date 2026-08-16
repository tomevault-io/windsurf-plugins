---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# RubricTrail maintainer rules

These repository-specific instructions apply to every AI-assisted change in
RubricTrail. They supplement, and do not replace, the generated Next.js block
above. Do not edit, move, weaken, or delete that generated block.

## Required orientation

Before making a change, inspect the current repository state and read the files
that define the affected behavior. At minimum, review these repository-wide
contracts when the task could affect them:

- `README.md`
- `SECURITY.md`
- `CONTRIBUTING.md`
- `docs/ARCHITECTURE.md`
- `docs/KNOWN_LIMITATIONS.md`
- `docs/TEST_REPORT.md`
- `.github/workflows/ci.yml`
- `.github/workflows/deploy-pages.yml`
- `package.json`

Also inspect the relevant implementation, all meaningful call sites, tests,
fixtures, schemas, migrations, and user-facing documentation before editing.
Do not rely on remembered behavior when the repository can be checked.

## Product mission and educational integrity

RubricTrail helps students:

- understand assignment and assessment requirements;
- trace rubric criteria back to source evidence;
- plan their own work;
- review progress;
- self-check whether requirements have been addressed.

The governing principle is: **Help students do the work, not replace the
work.**

Do not silently turn RubricTrail into an essay generator or answer-generation
product. Do not add functionality that:

- generates complete student submissions;
- invents rubric criteria;
- invents evidence, citations, sources, facts, or grades;
- claims that a student satisfies a criterion without evidence;
- encourages bypassing academic-integrity requirements.

Any substantial change to this educational-integrity boundary requires explicit
maintainer approval.

## Local-first and privacy invariants

The public GitHub Pages version is intended to remain local-first. Unless a task
explicitly concerns the optional self-hosted Live AI mode:

- coursework, drafts, rubric text, evidence, and uploaded files must not be sent
  over the network;
- full uploaded source documents must not be persistently stored;
- analytics or telemetry must not contain filenames, assignment titles,
  excerpts, rubric text, drafts, or other student free text;
- OpenAI credentials must never be exposed client-side;
- Live API code, routes, endpoints, credential markers, and runtime
  configuration must not leak into the public static artifact.

Do not weaken runtime validation, file-parsing safety limits, backup/import
validation, local-state validation, static-demo privacy audits, Live route
guards, request-size limits, or security checks merely to make a test pass.

Treat uploaded files, parsed external documents, `localStorage`, imported
backups, HTTP request bodies, external API responses, and AI/model output as
untrusted input. Validate them at their trust boundaries.

## Live AI boundary

The public static demo must remain usable without Live AI. Do not make Live AI
publicly accessible to arbitrary users without an explicit maintainer decision
and a separate production-readiness design covering at minimum:

- real user or session identity;
- authorization;
- durable rate limiting;
- budget and cost limits;
- abuse controls;
- an operator kill switch;
- explicit data-transfer consent;
- logging minimization;
- privacy documentation.

A shared bearer token is not a complete public multi-user authentication
system. Do not describe `store: false` as guaranteed zero data retention.

## Change discipline

One task should solve one clearly defined problem. Before editing:

1. explain the current behavior;
2. state the exact problem;
3. identify the invariants that must not change;
4. choose the smallest safe implementation;
5. identify the verification needed for the affected risk boundary.

Do not perform unrelated cleanup or refactoring. Do not expand scope merely
because another improvement is nearby. If a requested change reveals a separate
problem, report it as follow-up work instead of silently including it.

Large architectural changes, new dependencies, authentication systems,
state-management frameworks, persistence-model changes, privacy-policy changes,
educational-behavior changes, or public AI-service changes require explicit
maintainer approval.

Preserve user changes in a dirty worktree. Stage explicit intended paths rather
than broad, catch-all additions. Do not merge, release, deploy, tag, publish, or
modify external repository settings unless the maintainer has explicitly
authorized that action for the current task.

## AI-assisted development safety

This repository is maintained with substantial AI/Codex assistance, so

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sion612/rubrictrail](https://github.com/Sion612/rubrictrail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
