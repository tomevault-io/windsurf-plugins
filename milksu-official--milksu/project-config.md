---
trigger: always_on
description: Before changing anything, read:
---

# MilkSU Repository Guidance

## Start Here

Before changing anything, read:

1. `docs/developer/current-objectives.md`;
2. `docs/developer/document-status.md`;
3. `docs/architecture/current-system.md`;
4. the current Git branch, HEAD and working tree.

M3 product-loop work was squash merged to `main` on 2026-08-05. Continue from
`current-objectives.md`, current code, tests and Git history rather than reopening the merged PR, retired
ledgers or old sprint gaps. Build the next bounded slice, record adjacent non-blocking bugs near the relevant
code or current target notes instead of fixing them opportunistically, and fix immediately only when a problem
blocks the selected slice, threatens data/credentials/scope/private-remote boundaries, or invalidates the
acceptance result.

Do not use old milestones, ADR follow-ups, dated reviews, checkpoints, research notes or design audits as
an implementation queue. `docs/developer/development-plan.md` does not exist and must not be recreated.

## Product Code Admission

Read `docs/developer/product-code-admission.md` at these decision points:

- before designing a new product capability, public Desktop RPC API, persisted state, Sidecar resource or
  feature flag;
- before implementing Agent Harness behavior, compatibility/migration logic, experimental product
  surfaces or a capability already owned by Pi or another reviewed upstream component;
- before adding smoke, fixture, benchmark, browser-preview or release-acceptance infrastructure;
- during review and delivery closeout when deciding whether an implemented but unverified capability
  should remain in the production dependency graph.

Apply its four gates: design admission, development boundary, test/acceptance separation and
review/retention. Small fixes and documentation-only work do not require rereading it unless they change
one of those decisions.

## Collaboration

- Communicate with the user in Chinese unless they ask otherwise.
- Do not use emoji in code, comments, documentation, UI text or commit messages.
- Explain relevant Agent Harness concepts when they materially help product decisions or the user's
  interview and presentation preparation.

## User-visible language

Product UI, chat bubbles, notices, errors, slash descriptions, button titles and tool-result
`detail` text are for the user. Do not put agent-only implementation notes, harness comments,
internal thresholds, or “this is not X / 不拦手动” explanations in that copy. Keep those facts
in AGENTS.md, current-objectives, code comments, or model-only guidance such as
`codingWorkspaceGuidance()`.

Do not fill empty states with optional commentary, coaching, or “还没有 / 打开以后会出现”
status copy. If the control is missing a value, leave the surface blank and show only the
control’s own label, such as “选择项目”. The Coding new-conversation canvas may show the
product heading “我们要构建什么” or “我们在 {project} 中构建什么”.

## Product UI Color Boundary

- Current visual contract: [docs/design/current-visual.md](docs/design/current-visual.md).
  ak-ui tokens and scene CSS in `app/src/styles/`. Materials are graphite command surfaces, paper
  facts, cyan and gold. Acid green does not enter the product. Felinic stays for Vue behavior.
- Do not restore the old blue-black style, the retired tactical-archive / acid-green contract, paper
  or carbon textures, Showcase character art, sanity bars or 3D menus. `docs/design/milksu-game-ui-system.md`
  and `design-qa.md` are deleted and must not be recreated as current rules.
- Night mode uses graphite without an obvious blue, green or brown cast; day mode keeps paper neutrals.
  Cyan is current module and primary actions. Gold is secondary emphasis and the current focus bar.
  Success green only means success. Blue is reserved for links and explicit execution or diagnostic states.
- CTF, CVE, 实验室 and Coding tabs use the same charcoal-and-cyan system. Do not use `--info`, blue borders
  or blue-filled surfaces to distinguish those product modules.

## Beta Self-Bootstrap Boundary

- `MilkSU Beta.app` exists only for MilkSU's own self-bootstrap loop, where a Stable MilkSU reviewer controls
  an independently identified Beta build and verifies its branch, commit, tracking ID and user-visible task.
- Codex must not build or refresh the Beta app during ordinary implementation, debugging, UI validation or
  release preparation. Use unit/component tests, Sidecar tests, browser previews, or the Stable development
  runtime instead.
- Build Beta only when the user explicitly asks to run a MilkSU self-bootstrap exercise. A request to test a
  feature, inspect the desktop UI, or package a normal app is not self-bootstrap authorization.

## Current Product Boundary

MilkSU is an Electron/Chromium + Vue desktop app with a supervised Go Runtime and Pi Sidecars.

TokenFlux API traffic must use `https://tokenflux.dev/v1`. Never use the `tokenflux.ai` domain in product
code, configuration, test defaults or documentation.

- Pi owns the generic model session, context compaction and tool loop.
- MilkSU owns desktop authorization, workspace and credential boundaries, event projection and product UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MilkSU-Official/milksu](https://github.com/MilkSU-Official/milksu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
