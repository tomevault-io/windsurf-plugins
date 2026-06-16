---
trigger: always_on
description: - Product requirements live in `docs/brainstorms/`
---

# PwrAgent Repository Guidance

## Source of Truth

- Product requirements live in `docs/brainstorms/`
- Implementation plans live in `docs/plans/`
- UI theme tokens and visual language live in [docs/UI-THEME.md](docs/UI-THEME.md)
- Desktop UI direction lives in [docs/design/desktop-style-guide.md](docs/design/desktop-style-guide.md)
- The PwrAgent v2 design source bundle (HTML/CSS/JSX prototypes + chat transcripts) lives in [docs/design/pwragent-v2/](docs/design/pwragent-v2/) — see [docs/design/pwragent-v2/SOURCE.md](docs/design/pwragent-v2/SOURCE.md) for provenance and the "reference, not copy verbatim" policy
- The operator-facing site at <https://docs.pwragent.ai> lives in its own repo at [pwrdrvr/docs.pwragent.ai](https://github.com/pwrdrvr/docs.pwragent.ai) (split out from this repo on 2026-05-25). Edit there for per-platform setup walkthroughs, streaming/webhook explainers, and Settings → Messaging reference content. Contributor-facing messaging docs stay in [docs/messaging-*.md](docs/).

## Workflow

- Treat plan documents as decision artifacts, not implementation scripts.
- Keep changes aligned with the current active plan unless the user explicitly changes scope.
- Do not delete or "clean up" files in `docs/brainstorms/`, `docs/plans/`, or future `docs/solutions/` directories. **Don't rewrite plan / brainstorm / solution files that weren't created on your current branch either** — they are point-in-time decision artifacts, a historical record of what was decided when. The one exception: the plan file your current branch is executing is fair game for in-flight updates (progress checkboxes, deferred-to-implementation answers resolved as you work). The root [`.rgignore`](.rgignore) skips all three directories from default `rg` searches; override per-search with `rg --no-ignore` (or `rg -u`) when you actually want to grep them. Full rules in [`docs/plans/AGENTS.md`](docs/plans/AGENTS.md).
- GitHub Actions labels that intentionally trigger workflow behavior are
  documented in [.github/workflows/README.md](.github/workflows/README.md).
  Check that list before adding or using a CI-triggering PR label.
- Exclude `apps/desktop/.local/protocol-captures/` from broad searches by default. Only search it when the task is specifically about captured E2E protocol snippets.
- Never read Codex-owned storage files directly from PwrAgent code. Treat
  Codex session JSONL files, rollout files, and Codex sqlite databases as
  private implementation details; use the Codex App Server protocol instead.
  PwrAgent-owned files under `~/.pwragent/` and repo-local test fixtures are
  fine when the feature explicitly owns them. CI enforces common cases with
  `pnpm lint:codex-storage`; do not bypass that check by renaming variables or
  shelling out.
- Use the project-local [desktop E2E fixture seeding skill](.agents/skills/desktop-e2e-fixture-seeding/SKILL.md) when seeding or refreshing desktop replay fixtures from live captured sessions.
- For reliable desktop E2E runs, prefer `pnpm test:desktop-e2e` from the repo root. The package-level `pnpm --filter @pwragent/desktop test:e2e` path is also safe now because it builds `apps/desktop/out/` before launching Playwright.
- For manual screenshots of the branch-drift dialog, run `pnpm --filter @pwragent/desktop inspect:e2e:branch-drift`; it opens a replay-backed Electron fixture and waits until you close the app.
- To regenerate the README screenshots under `docs/assets/screenshots/`, run `pnpm --filter @pwragent/desktop screenshot:readme`. The full walkthrough (spec, fixtures, state-seeding helpers, native capture utilities) lives in [apps/desktop/AGENTS.md](apps/desktop/AGENTS.md) under "Capturing README Screenshots". macOS Screen Recording permission is required for whichever terminal/IDE runs the spec.
- When focusing root Vitest runs through `pnpm test`, pass file paths or filters directly, for example `pnpm test packages/agent-core/src/__tests__/overlay-store.test.ts`. Do not insert a standalone `--` before the focus args; `pnpm test -- packages/...` makes Vitest run the full workspace suite.

## Agent Instruction Files

- Keep a sibling `CLAUDE.md` symlink next to every `AGENTS.md`, pointing at that `AGENTS.md`, so Codex and Claude read the same local guidance.

## Pull Requests

- Use Conventional Commit-style PR titles: `type(scope): short description`.
- Prefer scopes that match the project area being changed:
  - `messaging` for Telegram, Discord, adapters, and messaging integrations.
  - `desktop` for the desktop app itself.
  - `agent-core` for the coding agent, currently the Grok coding agent.
  - `release` for packaging, signing, notarization, distribution, and auto-update pipeline.
  - `docs` for documentation changes.
  - `tests` for test coverage, fixtures, and test infrastructure.

## Release / Distribution

- The desktop release pipeline (Mac, signing, notarization, auto-update) is
  documented in [docs/desktop-release-runbook.md](docs/desktop-release-runbook.md).
- The Phase 1 → Phase 2 distribution channel migration runbook lives at
  [docs/desktop-distribution-phase-2-runbook.md](docs/desktop-distribution-phase-2-runbook.md).
- PwrAgent is MIT-licensed, owned by PwrDrvr LLC. Treat the repo-root

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pwrdrvr/PwrAgent](https://github.com/pwrdrvr/PwrAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
