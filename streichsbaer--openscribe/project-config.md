---
trigger: always_on
description: - Load `SOUL.md` at the start of work.
---

# OpenScribe Agent Rules

## SOUL

- Load `SOUL.md` at the start of work.
- Always read `SOUL.md` before planning or editing code in each new session.
- Treat `SOUL.md` as the product and engineering voice for this repository.
- Use `SOUL.md` to guide priorities, privacy stance, and communication tone.

## Build Policy

- Implement the current behavior directly.
- Do not add migration code, deprecation handling, compatibility shims, fallback paths, or legacy settings upgrades unless the user explicitly requests them.
- Prefer a single clear path that works now over backward-compatibility logic.

## Completion Standard

- Complete the full requested outcome when the next execution step is available.
- Do not stop after an intermediate milestone such as local build success, artifact generation, or PR creation when release, publication, merge, or verification steps are still actionable.
- If an external gate remains, keep driving until the gate is resolved or report the exact blocker, current state, and next command needed.
- For release and distribution work, verify the published state after the final write operation instead of assuming success from a prior local step.

## Writing Style

- Do not use em dashes.
- Do not use contrastive negation phrasing such as "it is not X, it is Y."

## Documentation Boundaries

- `SOUL.md` captures enduring product values, priorities, and quality intent.
- `AGENTS.md` captures repository workflow, execution rules, and collaboration protocol.
- `site-docs/product/spec.md` and `site-docs/product/roadmap.md` are canonical product docs.
- `site-docs/ops/` contains operational runbooks for testing, releases, and issue tracking.
- Update the canonical location first and replace duplicate content with links.

## CI Security and Review Hygiene

- Pin third-party GitHub Actions to immutable full commit SHAs.
- Pin externally downloaded CI dependencies to explicit versions.
- Use least-privilege workflow permissions and scope write or OIDC permissions to the smallest set of jobs.
- For public repositories, avoid running privileged jobs for fork pull requests.
- Triage external review findings before merge and either fix them or document rationale for deferral.

## Product Specs Source of Truth

- Use `site-docs/product/spec.md` as the canonical product spec.
- Use `site-docs/product/roadmap.md` as the canonical roadmap summary.
- When asked what is next, start from `site-docs/product/roadmap.md`.
- Follow links from product docs into deeper pages under `site-docs/`.

## Issue and Feature Tracking

- Use GitHub Issues as the live tracker for features, bugs, and docs work.
- Before creating, editing, labeling, commenting on, or closing any issue, load `site-docs/ops/issue-tracking.md` and `site-docs/ops/label-conventions.md`.
- Follow `site-docs/ops/issue-tracking.md` for tracking workflow and saved query links.
- Follow `site-docs/ops/label-conventions.md` for label taxonomy.
- New issues should include one `type/*`, one `status/*`, and one `area/*` label.
- Keep product docs aligned with issue status when behavior changes.
- When work is completed, set `status/done` and close the related issue.
- For GitHub CLI issue comments and close comments, use multiline input via heredoc with `--body-file -` or `$(cat <<'EOF' ... EOF)`.
- Do not include literal `\n` escape sequences in issue comments.
- Use issue-first external collaboration. Redirect unsolicited external pull requests to issues.

## Change Approval

- Stefan is the final approver for changes to `SOUL.md`, `AGENTS.md`, `site-docs/product/spec.md`, and `site-docs/product/roadmap.md`.
- I can draft and apply updates when Stefan requests them directly.
- If I identify a governance improvement outside a direct request, I propose it first, then wait for approval before editing.
- Product direction changes require explicit approval before implementation.

## Release Verification

- Follow `site-docs/ops/testing.md` and `site-docs/ops/release.md` for release validation steps.
- Keep release checklists in `site-docs/ops/` and avoid duplicating them in `SOUL.md`.

## Verification Playbooks

- For app behavior changes or test requests, load `site-docs/ops/testing.md` before running checks.
- For docs content, docs styling, docs workflow, or GitHub Pages verification, load `docs/ops/docs-verification.md` and `.agents/skills/docs-visual-review/SKILL.md`.
- For app UI, docs layout, or docs image changes, verify both a compact laptop-sized viewport and a larger viewport. Confirm key actions stay visible or reachable by scrolling, and confirm required image detail is not cropped away.
- For Cloudflare DNS, edge security headers, HTTPS settings, or domain routing tasks, load `docs/ops/cloudflare.md` before making changes.
- For GitHub issue triage or issue updates, load `site-docs/ops/issue-tracking.md` and `site-docs/ops/label-conventions.md` before running issue commands.
- For post-push docs verification, run `$docs-visual-review --remote-url https://openscribe.dev/ --out artifacts/docs-visual/remote-latest` and report `report.md` plus screenshot paths.
- For docs visual checks, inspect generated screenshots directly with the image tool. Do not run auxiliary image processing commands unless Stefan explicitly requests deeper analysis.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [streichsbaer/openscribe](https://github.com/streichsbaer/openscribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
