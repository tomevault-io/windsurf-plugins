---
trigger: always_on
description: Telegraph style. Root rules only. Read scoped `AGENTS.md` before subtree work.
---

# AGENTS.MD

Telegraph style. Root rules only. Read scoped `AGENTS.md` before subtree work.
Skills own workflows; root owns hard policy and routing.

## Start

- Repo: `https://github.com/openclaw/openclaw`
- Replies: repo-root refs only: `extensions/telegram/src/index.ts:80`. No absolute paths, no `~/`.
- Docs/user-visible work: `pnpm docs:list`, then read relevant docs only.
- Fix/triage answers need source, tests, current/shipped behavior, and dependency contract proof.
- Reviews/answers: high confidence required. Default to exhaustive relevant codebase search/read, including owners, callers, siblings, tests, docs, and upstream/dependency contracts before verdict. Diff-only review is insufficient.
- Review default: read the whole changed function/module plus callers, callees, sibling implementations, adjacent tests, scoped docs, and dependency/Codex contracts before saying `good`, `bad`, `best fix`, `proof sufficient`, or posting a comment. If challenged, keep reading first; do not defend the earlier verdict until the missing path is checked.
- Dependency-touching work: direct dependency inspection is mandatory when feasible; do not rely on assumptions, wrappers, or memory. Most dependencies are OSS, so read their source/docs/types. Codex-related work has a hard gate: the acting agent must personally inspect sibling `../codex` source for the exact protocol/runtime behavior before any verdict, comment, approval, merge recommendation, code change, or `proof sufficient` claim. If missing, clone `https://github.com/openai/codex.git` there first. Subagent reports, PR text, OpenClaw wrappers, generated schemas, memory, and prior bot reviews do not satisfy this gate. No direct `../codex` check means no Codex verdict. Cite Codex files/lines checked in final/review/comment.
- Dependency-backed behavior: read upstream docs/source/types first. No API/default/error/timing guesses.
- External API work: live test required. Google/search for additional proof. Prefer official docs/source/types; cite current proof. No memory-only API claims.
- Live-verify when feasible. Never print secrets.
- Missing deps: `pnpm install`, retry once, then report first actionable error.
- CODEOWNERS: maint/refactor/tests ok. Larger behavior/product/security/ownership: owner ask/review.
- Product/docs/UI/changelog wording: "plugin/plugins"; `extensions/` is internal.
- New channel/plugin/app/doc surface: update `.github/labeler.yml` + GH labels.
- New `AGENTS.md`: add sibling `CLAUDE.md` symlink; edit `AGENTS.md` only.

## ClawSweeper Review Policy

- OpenClaw-specific review rules live here; generic ClawSweeper prompts stay repo-agnostic.
- ClawSweeper-owned schema, labels, close reasons, protected-label gates, maintainer-item gates, and mutation rules live in `openclaw/clawsweeper`.
- Review workers read this full root `AGENTS.md` before judging; no reliance on search snippets, `head`, partial ranges, local excerpts, or truncated copies. Then read every scoped `AGENTS.md` that owns touched paths.
- Optional integrations, providers, channels, skill bundles, MCP surfaces, and service workflows route to plugins, ClawHub, or owner repos when current seams suffice. Keep core items for missing core/plugin APIs, bundled regressions, security/core hardening, or maintainer product decisions.
- Plugin APIs, provider routing, auth/session state, persisted preferences, config loading, config/default additions, migrations, setup, startup checks, and fallback behavior are compatibility/upgrade-sensitive. Treat config breaks, new config/default surfaces, removed fallbacks, fail-closed changes, stricter validation, or new operator action as merge risk even with green CI when they can affect existing users, upgrades, provider/plugin behavior, or maintainer operations.
- For PRs that add, remove, or change config/default surfaces with possible compatibility, upgrade, provider/plugin, operator, setup, startup, or fallback impact, ClawSweeper review should emit a `reviewMetrics` entry when practical. The metric should name the count and direction of the changes, such as added, changed, or removed config/default surfaces, and explain why the metric matters before merge. When the metric indicates concrete merge risk, also surface the concern in `risks`, use `mergeRiskLabels` when the risk matches the label rubric, make `bestSolution` name the desired pre-merge state, and ensure `labelJustifications` explain the specific reason rather than restating the label.
- Review whole decision surfaces, not only the touched runtime, provider, channel, harness, plugin seam, or context path. Check sibling Codex/Pi-style runtimes, provider/model routing, channel delivery, gateway/protocol, plugin SDK, and context-management paths when relevant.
- Every PR review must explicitly ask whether the PR is the best fix, not merely a plausible fix. Verdicts need a best-fix judgment backed by enough code reading to compare owner boundaries, callers, siblings, tests, docs, current `main`, shipped behavior when relevant, and dependency/Codex contracts when involved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vignesh2027/Vilvona-AI](https://github.com/vignesh2027/Vilvona-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
