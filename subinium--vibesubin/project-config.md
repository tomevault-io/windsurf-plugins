---
trigger: always_on
description: Read this file first in every new session. It encodes the operational rules that apply to this repo. The pack-level invariants live in [`docs/PHILOSOPHY.md`](./docs/PHILOSOPHY.md); the update cadence lives in [`MAINTENANCE.md`](./MAINTENANCE.md); the user-facing overview lives in [`README.md`](./README.md). This file is for the rules that have to be checked *every session*.
---

# vibesubin — AI operator guide

Read this file first in every new session. It encodes the operational rules that apply to this repo. The pack-level invariants live in [`docs/PHILOSOPHY.md`](./docs/PHILOSOPHY.md); the update cadence lives in [`MAINTENANCE.md`](./MAINTENANCE.md); the user-facing overview lives in [`README.md`](./README.md). This file is for the rules that have to be checked *every session*.

## 🛑 Never do

1. **Never rewrite the root `README.md` wholesale.** Targeted edits only — skill table, direct-call list, workflow bullets, accuracy fixes. Never restructure sections, reorder content, or rewrite prose for style. Same rule applies to `README.ko.md`, `README.ja.md`, `README.zh.md`. End-to-end rewrites of translations are acceptable only when the structural scope of the change (skill split, new skill, rename) makes surgical edits impossible — and even then, preserve the existing voice.
2. **Never add a new worker skill past the 10 + 1 category cap.** The pack caps at **10 code-hygiene workers** plus **1 process worker**, for 11 total (the `vibesubin` umbrella is not counted). As of v0.4.0 all 11 slots are used. Code hygiene (10): `refactor-verify`, `audit-security`, `fight-repo-rot`, `write-for-ai`, `setup-ci`, `manage-secrets-env`, `project-conventions`, `manage-assets`, `unify-design`, `codex-fix`. Process (1): `ship-cycle`. Any future new capability must extend, split, or displace an existing skill within its category — do not expand either cap. The two-bucket split exists because process work (issue/release orchestration) is a distinct cognitive category from code hygiene; users don't confuse them.
3. **Never ship a `SKILL.md` over 500 lines.** Enforced by `scripts/validate_skills.py`. Extract tail sections into `references/*.md` and replace with one-line links from `SKILL.md`. Progressive disclosure is load-bearing — long `SKILL.md` files get partially read by Claude Code.
4. **Never ship a worker skill without a "Harsh mode — no hedging" section.** Every worker the umbrella launches must implement the `tone=harsh` marker check. Partial coverage is the root cause of *"harsh mode doesn't feel harsh"*. Balanced-mode silent fallback is a regression.
5. **Never claim a task is done without `python3 scripts/validate_skills.py` passing.** The validator is the contract between what `SKILL.md` promises and what exists on disk. A failure is a ship blocker — not a warning.
6. **Never commit `.env`, credentials, marketplace tokens, or SSH private keys.** `.gitignore` covers the usual suspects; verify `git ls-files | grep -iE '\.env$|\.pem$|id_rsa'` before every release commit.
7. **Never skip the `sweep=read-only` marker in the umbrella's parallel launch block.** The 6 editable worker specialists (`refactor-verify`, `setup-ci`, `write-for-ai`, `manage-secrets-env`, `project-conventions`, `unify-design`) rely on it to stay read-only during `/vibesubin` sweeps. Without the marker, they fall back to full edit behavior, which is incorrect for a sweep.
8. **Never bump the plugin version in only one manifest.** `.claude-plugin/marketplace.json` and `plugins/vibesubin/.claude-plugin/plugin.json` must both change together. `plugin.json` has been stale before — catch it in review.

## ✅ Always do

1. **Run `python3 scripts/validate_skills.py` after any skill edit.** This is the before-you-commit check. Verification command: `python3 scripts/validate_skills.py`. Expected output: `OK — every promise in N skills resolves to an actual file`.
2. **Update `CHANGELOG.md` in functional-only style.** Rule: every bullet describes an observable change. No narrative, no meta-rationale, no "we decided to", no emotional framing. If you're explaining *why*, the bullet belongs in a commit body or release notes, not the CHANGELOG.
3. **Update all four READMEs together when a skill is added, renamed, or deleted.** Surgical edits only to `README.md`, `README.ko.md`, `README.ja.md`, `README.zh.md` — skill table rows, direct-call list, workflow bullets, "never edit" phrasing, section headings. Korean/Japanese/Chinese users see the same structure as English, in natural voice for their language.
4. **Sync plugin version across both manifests in the same commit.** `marketplace.json` is the canonical source; `plugin.json` mirrors it. Description text should also stay in sync.
5. **Verify harsh-mode coverage before shipping a new worker.** Every worker that can receive the `tone=harsh` marker must have a "Harsh mode — no hedging" section in its `SKILL.md` that (a) checks for the marker, (b) switches output rules to direct / no-hedging framing, (c) preserves factual accuracy — never inflates severity, never invents findings.
6. **Respect the 4-part output shape in every worker.** Every skill's output follows: what it did, what it found, what it verified, what you should do next. This is load-bearing for the umbrella's synthesis step.

## 🚀 Release process


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [subinium/vibesubin](https://github.com/subinium/vibesubin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
