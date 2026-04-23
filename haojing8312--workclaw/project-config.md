---
trigger: always_on
description: - `apps/runtime/` contains the React desktop app shell, frontend flows, and UI tests.
---

# AGENTS.md instructions for E:/code/yzpd/workclaw

## Project Overview
- `apps/runtime/` contains the React desktop app shell, frontend flows, and UI tests.
- `apps/runtime/src-tauri/` contains the Tauri backend, desktop integrations, and Rust integration tests.
- `apps/runtime/sidecar/` contains the sidecar runtime, adapters, browser automation bridge, and sidecar tests.
- `packages/*` contains shared Rust crates for routing, policy, models, executor, skill packaging, and runtime support.
- Root `package.json` commands are the source of truth for local verification and release-sensitive checks.

## Repo-Local Workflow Skills
- `$workclaw-implementation-strategy`: Use before editing runtime behavior, routing, provider integration, tool permissions, sidecar bridge behavior, or vendor sync boundaries.
- `$workclaw-change-verification`: Use when changes affect code, tests, builtin skill assets, or build/test behavior before claiming the work is complete.
- `$workclaw-release-readiness`: Use when changes affect versioning, release documentation, installer branding, packaging outputs, or vendor release lanes before deciding a branch is safe to ship.
- `$workclaw-release-prep`: Use before publishing to recommend the next version and draft bilingual Chinese + English release notes for confirmation.
- `$workclaw-release-publish`: Use only after version and release notes are confirmed, to update release metadata, push the release tag, and generate local desktop artifacts.

## Repo Hygiene Governance

- Treat orphan files, dead code, stale docs, duplicate implementations, and temporary artifacts as a maintenance surface, not a one-off cleanup task.
- Prefer repo hygiene review before deletion. Do not remove suspicious files or code only because they appear unused in one static signal.
- Use `pnpm review:repo-hygiene` for non-blocking repo hygiene reporting when the task is cleanup-focused or when a large feature leaves likely follow-up debris.
- Use focused repo hygiene subcommands when one narrow signal is enough:
  - `pnpm review:repo-hygiene:deadcode`
  - `pnpm review:repo-hygiene:artifacts`
  - `pnpm review:repo-hygiene:drift`
  - `pnpm review:repo-hygiene:dup`
  - `pnpm review:repo-hygiene:loc`
  - `pnpm review:repo-hygiene:cycles`
- Treat duplicate implementations, oversized files, and import cycles as review-first governance signals. They should trigger triage and split plans, not blind deletion or mechanical rewrites.
- Use `workclaw-repo-hygiene-review` to classify candidates and recommend the smallest safe cleanup batch before destructive edits.
- Use `workclaw-cleanup-execution` only after review selected a cleanup batch and its reviewed action per file.
- Cleanup changes still require `workclaw-change-verification` when code, tests, docs, or skill files change.
- Treat generated, runtime-owned, dynamically discovered, or config-driven files as high-risk cleanup surfaces unless a rule explicitly marks them safe.

## Current Project Stage
- WorkClaw is currently an early-stage open source project with a single primary maintainer.
- Default development may happen directly on `main` when that is the most practical path.
- Repo-local skills are lightweight self-check and workflow guidance tools, not mandatory PR approval gates.
- PR-based review, automated merge gates, and stricter branch policies are optional future upgrades, not the default workflow today.

## Mandatory Skill Usage
- Use `$workclaw-implementation-strategy` before changing runtime behavior, routing, provider integration, tool permissions, sidecar protocols, IM orchestration behavior, or vendor sync boundaries.
- Use `$workclaw-change-verification` when changes affect runtime code, tests, examples, builtin skills, or build/test behavior. Do not claim completion until the relevant checks have actually run.
- Use `$workclaw-release-readiness` when changes affect versions, release docs, installer branding, packaging, or vendor release lanes.

These skills should be treated as lightweight guardrails for the maintainer's own workflow. They do not imply that every change must go through a PR or a separate human approval step.

## Build And Test Commands
- Runtime dev: `pnpm app`
- Desktop build: `pnpm build:runtime`
- Sidecar tests: `pnpm test:sidecar`
- Rust fast path: `pnpm test:rust-fast`
- Runtime E2E: `pnpm test:e2e:runtime`
- Builtin skills: `pnpm test:builtin-skills`
- Real agent evals: `pnpm eval:agent-real --scenario <id>`

## Release-Sensitive Commands
- Version checks: `pnpm release:check-version`
- Release tests: `pnpm test:release`
- Installer checks: `pnpm test:installer`
- Release docs: `pnpm test:release-docs`
- Vendor lane checks: `pnpm test:openclaw-vendor-lane`
- Packaging sanity: `pnpm build:runtime`

## Compatibility And Safety Rules
- Preserve existing user-visible runtime behavior unless the change is intentional and called out explicitly.
- Treat packaging, installer, release docs, and vendor sync changes as release-sensitive, not ordinary code edits.
- Prefer the smallest command set that proves the touched area is verified, but never skip a required check for the changed surface.
- Verification claims must cite the commands actually run and whether any areas remain unverified.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haojing8312/WorkClaw](https://github.com/haojing8312/WorkClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
