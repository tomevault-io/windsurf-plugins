---
trigger: always_on
description: Decision-ready improvement programs for neodx. A plan here is the source of truth for what a
---

# neodx plans

Decision-ready improvement programs for neodx. A plan here is the source of truth for what a
multi-step effort intends, what is locked, and where it stopped.

The full S0–S7 program ledger lived in the Nubis checkout while the polygon was open; it closed
2026-08-22 (Vite+ and TS references **deferred**). Remaining cross-repo state lives in the Nubis
plans index (`/Users/host/WebstormProjects/nubis/.agents/plans/AGENTS.md`) and named GitHub issues.
This folder holds neodx-local status, evidence pointers, and future neodx-originated plans.

Session rules live in [`../../AGENTS.md`](../../AGENTS.md): standing branch `work`,
`yarn changeset add` for changelogs, **publish freeze until a consolidated-release command**
after the #180 land, session-close gate, slug-first names, and **tighten what you touch**.

## How to use

- Create a plan when work spans more than a single change and needs locked decisions across sessions.
- One file per program, named `YYYY-MM-DD-<slug>.md`.
- Keep the **decision table** and **progress ledger** inside the plan.
- Close a plan (delete, or keep with a reason) only when slices are verified and debt is recorded.

## Program status (revalidated 2026-08-25)

Canonical `origin/main` is **`cf60915`** (Version Packages [#185](https://github.com/secundant/neodx/pull/185)).
`strip-source-bridges` ([#180](https://github.com/secundant/neodx/issues/180)) is **closed**: npm
**1.1.1**, packuments have no `development` / `./src` bridges. Further publishes wait for a
**consolidated-release** command. Oxlint `typeCheck` and later leftovers stay on `work`; they were
not part of this npm land. Re-read `git rev-parse --short origin/main` before citing SHAs.

All nine publishable packages are on npm **1.1.1** with OIDC provenance:

- **1.0.0** honesty freeze (run [31811688371](https://github.com/secundant/neodx/actions/runs/31811688371));
- **1.0.1–1.0.3** packaging fixes — workspace-protocol rewrite in published manifests
  (PRs [#173](https://github.com/secundant/neodx/pull/173)/[#175](https://github.com/secundant/neodx/pull/175)/[#177](https://github.com/secundant/neodx/pull/177));
- **1.1.0** paired-dts pack contract (PR [#172](https://github.com/secundant/neodx/pull/172), merged `50c6b50`):
  paired `.d.mts`/`.d.cts`, types-first `exports`, required `attw` CI gate after pack —
  [#164](https://github.com/secundant/neodx/issues/164) closed. Registry-verified:
  `attw --from-npm --profile node16 @neodx/std@1.1.1` is green.

The S0–S7 program is closed.

| Stream                   | Final status                                                                                                      | Evidence                                                                                                                                                                               |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| S0 Honesty + graph       | Done                                                                                                              | PR #160 history                                                                                                                                                                        |
| S1 Dep chunks C1–C3      | Done                                                                                                              | `renovate` still open                                                                                                                                                                  |
| S2 Vite+ migrate (WP-V2) | Done                                                                                                              | [Vite+ report](../reports/vite-plus-migration.md), [Oxlint delta](../reports/oxlint-eslint-kit-delta.md)                                                                               |
| S3 Solidify              | `PASS_WITH_DEBT` — verify-exports, publint, attw, SECURITY, OIDC provenance shipped; #162/#163 + C8 open          | [#162](https://github.com/secundant/neodx/issues/162)/[#163](https://github.com/secundant/neodx/issues/163)                                                                            |
| S4 AI meta               | AGENTS/CONTRIBUTING/workflows remain; imported skills **withdrawn**                                               | [workflows/index.md](../workflows/index.md)                                                                                                                                            |
| S5 TS project references | Honesty end-state + typeAware; `typeCheck` stays off until `oxlint-typecheck`                                     | [before](../reports/ts-project-references-before.md) · [research](../reports/ts-project-references-research.md) · [implementation](../reports/ts-project-references-implementation.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secundant/neodx](https://github.com/secundant/neodx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
