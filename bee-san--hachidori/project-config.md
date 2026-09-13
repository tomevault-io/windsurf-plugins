---
trigger: always_on
description: These instructions apply to the entire repository.
---

# AGENTS.md

These instructions apply to the entire repository.

## Working agreement

- Deliver every repository change through a pull request from a dedicated branch. Never push changes directly to `main`. Do not stop at a local commit or branch: push it and open the pull request.
- Keep each pull request to one coherent outcome that can be understood, tested, and reviewed independently.
- Commit frequently in small, coherent units. Prefer a failing focused contract test followed promptly by its implementation commit when test-first work is practical.
- Prefer the smallest straightforward change that satisfies the request and fits the existing architecture.
- Do not mix requested work with drive-by refactors, renames, formatting churn, dependency updates, or unrelated cleanup.
- Deduplicate an invariant or algorithm when two runtime contexts genuinely need the same behavior; do not introduce a framework for a one-off.
- Read `CONTRIBUTING.md` and the relevant architecture or test documentation before changing an unfamiliar area.

## Scope, tests, and guards

- Implement the requested behavior rather than hypothetical adjacent requirements.
- Do not add speculative abstractions, fallbacks, compatibility layers, validation, resource limits, or safety guards. A new guard should address an explicit requirement, a reproducible failure, or a documented invariant, and its reason should be clear in the pull request.
- Dictionary archives are user-selected local inputs. Do not invent fixed limits for archive bytes, entry counts, expanded size, or compression ratios unless the task explicitly requires them. Preserve existing path and import-staging correctness boundaries.
- Do not add broad or extensive test coverage by default. Add a focused regression test when behavior changes or a bug needs to stay fixed; do not duplicate coverage already provided by a suitable suite.
- Avoid adding test-only dependencies or expanding fixtures unless the changed behavior genuinely needs them.

## Issue #9 scope and phases

- D1-D9 and E1-E27 are delivered. The user's subsequent request, "work on l2 to l5", authorizes L2 backup/restore, L3 per-dictionary update schedules, L4 lookup/corpus-seen statistics, and L5 definition blur as the current phase. Deliver them in focused pull requests preserving the completed dictionary and reader behavior, using GSM PR #549 as the reference.
- L2 (#62), L3 (#63) and L4 (#65) are merged; L5 is #67. When asked to "do L1-L5", the user confirmed L2-L5 only: L1 profiles stay excluded.
- The dictionary-only contract below limits dictionary-only tasks; it does not prohibit separately authorized E-series or L2-L5 work. L3 intentionally extends D6's original global-only schedule; L5 depends on L4. L1 profiles and L6 configurable/custom popup actions remain excluded, as does localization.

## Dictionary-only issue #9 contract

When implementing the dictionary-only scope from issue #9:

- For a dictionary-only task, D1-D9 are the complete scope. Do not pull E-series or Later features into that task; E-series work requires its own explicit authorization as recorded above.
- The issue owner's later comment overrides the original D6 prose: a scheduled update run checks and automatically installs available updates. Manual Check now still records availability without installing.
- Keep one global update schedule: Off, hourly, daily, weekly, or monthly. Do not add per-dictionary policy, hidden profiles, alternate backends, or due-time machinery.
- Preserve stable package IDs, canonical-title engine keys, order, alias, enabled/favourite state, groups, and trusted source metadata across reimports and updates. A package with several bank kinds remains one package.
- Use the exact five recommended sources: the four named by D2 plus Bee's Ultimate Grammar Dictionary, added on the issue owner's request. Install them sequentially, skip installed sources, continue after a failure, and retry only missing sources. Store catalogue metadata only after final-URL, repository, title, revision, index, and defining-capability validation.
- Keep shared catalogue/source trust rules in one native ES module used by every context that enforces them. Recommended sources remain catalogue-pinned; generic managed sources require complete HTTPS descriptors.
- A generic update index may provide a new HTTPS `downloadUrl`; otherwise retain the installed fallback. Validate the index and archive final URLs, and bind one checked update plan through download and commit.
- Bind a managed update to the exact package ID, generation path, installed revision, and source descriptor that was checked. Revalidate at the commit snapshot so a stale check cannot overwrite a concurrent reimport. Do not key this guard to the broad state revision because presentation-only edits may be preserved.
- Persist a successful update's `up-to-date` status in the same package CAS as the replacement. Apply check/failure status only if the checked generation still matches. Reject a managed title change that collides with another installed package.
- Dictionary groups use stable IDs. Names are unique after case/whitespace normalization and `All` is reserved. Group-only state writes must not invalidate lookups; rerenders must preserve deliberate keyboard focus.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bee-san/hachidori](https://github.com/bee-san/hachidori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
