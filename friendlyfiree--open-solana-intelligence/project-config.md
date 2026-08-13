---
trigger: always_on
description: This file applies to the entire repository. It is the persistent operating
---

# OSI Engineering Contract

This file applies to the entire repository. It is the persistent operating
contract for Codex and any other engineering agent working on Open Solana
Intelligence (OSI).

## 1. Mission and authority

OSI is a public-good intelligence platform that turns open-source and
open-chain material into attributable, wallet-signed, community-reviewed and
challengeable records. Its product value is process integrity, not automatic
truth, guilt, legal certainty, recovery, custody or guaranteed payment.

The accepted V2 blueprint on `main` is the implementation baseline. Use these
documents as the source of truth, in this order:

1. `docs/OSI_V2_PRODUCT_CONSTITUTION.md`
2. `docs/OSI_V2_DOMAIN_MODEL.md`
3. `docs/OSI_V2_STATE_MACHINES.md`
4. `docs/OSI_V2_ROLE_PERMISSION_MATRIX.md`
5. The remaining `docs/OSI_V2_*.md` specifications
6. The legacy V1 implementation, only as a compatibility reference

Read `docs/OSI_DELIVERY_BRIEF.md` before every task.

Do not silently weaken or reinterpret a product invariant. If two accepted
documents conflict, identify the conflict in the task report and implement the
safer, least-privileged option when it can be done without changing product
meaning. Otherwise keep writes disabled for the affected path and record the
decision needed. Do not restart a broad blueprint-review loop.

The following accepted implementation details must be made explicit in the
relevant implementation slice before its production writes are enabled:

- exact cutover and rollback delta;
- exact effects of an accepted challenge;
- nullable-state checks for resolutions;
- the maintainer initial-open path alongside analyst initial review;
- the server actor responsible for each class-A Memo anchor.

## 2. Repository facts

- Frontend: static `index.html`, modular CSS and classic JavaScript; no build
  system and no package manager manifest.
- Backend: Supabase PostgreSQL and Edge Functions under `supabase/functions/`.
- Existing dependency-free regression test:
  `node tests/xss-escaping.test.js`.
- V2 database changes belong in ordered, additive files under
  `supabase/migrations/` once that directory is introduced.
- The stable pre-V2 checkpoint is tag
  `v0.9.0-stable-pre-case-model` at commit `1491377`.

Always inspect the repository and relevant blueprint sections before editing.
Do not assume the README's legacy examples describe the V2 architecture.

## 3. Autonomous working rules

Routine, reversible engineering work is autonomous: inspect, edit, run tests,
create a task branch, commit and open a PR without asking the product owner to
make technical choices. Explain material deviations and security findings.

For scope-limited OSI tasks, standing user authorization also covers the full
safe delivery loop without another routine approval: branch, commit, push, PR,
merge after all required CI is green, dispatch of an existing reviewed
main-only production workflow, dry-run-approved additive migrations, deployment
of only the Edge Functions named by the task, dedicated feature-flag
enable/disable, and read-only post-deployment smoke verification. This standing
authorization never broadens task scope and applies only when the exact project
ref, current `main`, intended production impact, rollback/disable plan, and
task-limited diff have all been verified.

If a rollout or smoke check fails, fail closed by disabling only the affected
dedicated feature flag, preserve immutable data, identify the exact log-backed
root cause, prepare a focused forward-fix PR, and resume the existing workflow
at most once after that PR merges with green CI. Do not repeatedly retry a
failing production workflow. Authentication, branch protection, or a mandatory
interactive browser/2FA challenge are valid manual blockers; otherwise do not
hand routine GitHub or Supabase delivery steps back to the product owner.

The product owner is a domain expert in on-chain intelligence and forensics,
not a software engineer. Product, governance, threat-model and scope decisions
are theirs and are authoritative; implementation choices are yours. Reports
must use plain language and include exact commands or buttons only when a
manual step is genuinely unavoidable. Never report "working" or "complete"
from a narrative claim alone; prove it with a diff, test, query result or
deployment verification. See section 15 for what that division of labour
requires of you.

For every task:

1. Inspect current Git state and relevant files.
2. State the intended scope and production impact.
3. Make the smallest coherent implementation slice.
4. Test positive, negative and authorization paths in proportion to risk.
5. Inspect the final diff and ensure unrelated user changes are preserved.
6. Report the required handoff listed in section 14.

## 4. Git safety

- Start work from the current verified `main` and use a dedicated task branch.
- Never commit directly to `main`. A task-limited PR may be merged autonomously
  only when its required CI is fully green and branch protection permits it.
- Never use `git reset --hard`, destructive checkout, force push, history
  rewriting, or deletion of another contributor's work.
- Do not mix unrelated cleanup with a security or schema slice.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [friendlyfiree/open-solana-intelligence](https://github.com/friendlyfiree/open-solana-intelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
