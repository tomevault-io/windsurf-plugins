---
trigger: always_on
description: This file applies to the entire repository.
---

# AGENTS.md

This file applies to the entire repository.

`CONCEPTS.md` at the repo root holds the shared domain vocabulary — entities, named processes,
and the faction / Army of Renown / battle-formation hierarchy — relevant when orienting to the
codebase or discussing how army-building concepts relate. `docs/solutions/` holds documented
solutions to past problems (bugs, architecture patterns, workflow learnings), organized by
category with YAML frontmatter (`module`, `tags`, `problem_type`) — relevant when implementing
or debugging in documented areas.

## Project purpose and current state

AoS Reminders turns an Age of Sigmar army configuration into phase-ordered reminders.

This is an Age of Sigmar fourth-edition codebase:

- the browser runtime uses the canonical model under `src/aos4/`
- the checked-in runtime is generated from the accepted `aos4-corpus-2026-08-18` snapshot
- importing, cloud armies, and army sharing are AoS 4-native: roster parsers in
  `src/importers/` (official app text, Listbot text and file upload, Sigdex text, New Recruit
  `.ros`/`.rosz`/`.json`), roster resolution in `src/aos4/import/`, and the Auth0-authorized cloud
  client in `src/api/armyApi.ts`
- an army document may opt into Legends units (`allowsLegends`); Legends applies as an overlay on
  the document's rules context during selection resolution
- unrecognized browser state is deleted and replaced with a schema-valid army document; it is never
  translated
- the 27 decoded factions that field units are selectable through one source-complete relationship
  graph; the 28th row, `Endless Spells`, is a Wahapedia container for universal manifestations
  rather than an army, and generation now offers nothing from it, so `armyFactions` has nothing
  left to exclude
- manifestations are a category of unit rather than an army: the five universal manifestation lores
  and their 18 warscrolls come off the `Endless Spells` container page and are offered by all 27
  armies instead of by the container, through the review's `universalFactionContent` gate
- the accepted corpus contains 1,296 warscrolls, 1,012 battle profiles, 5,074 abilities,
  2,269 weapons, 1,493 content groups, and 20,084 live source records
- the 75 Regiments of Renown on the accepted collection pages are classified
  `regiment-of-renown` content groups (reviewed `regimentsOfRenown` input), offered by exactly
  their inclusion factions with their regiment abilities and member warscrolls linked
  (issue #1858)
- current standard, General's Handbook 2026-27 (`Scourge of Aqshy`), Spearhead, Legends, and
  historical rules contexts isolate parallel and retired records; the browser defaults to the
  current 2026-27 seasonal context
- strict generation consumes every live record and separately dispositions 19,113 superseded
  records (the May 2026 bulk warscroll/faction-rule rows plus the seventeen index-era Ogor
  datasheets the July 2026 battletome rewrites replaced, issues #1850 and #1880) so none can
  leak into runtime
- all 1,350 extracted official battle-profile facts have an explicit disposition: 1,013 apply to
  runtime, 1 remains a profile-only gap, 289 remain structured references, and 47 are superseded
- the earlier candidate/cohort reports remain checked-in reconnaissance history, not current
  blockers

Do not confuse these version numbers:

- `package.json` reports the application version (`6.0.0`)
- “AoS 4” means the Games Workshop game edition released in 2024
- the army-document and catalog schema versions are independent internal contracts

## Non-negotiable constraints

- `src/aos4/` may depend only on its own modules, Node built-ins, and third-party packages.
  Application code may depend inward on `src/aos4/`; the domain layer must not depend outward.
- Keep source acquisition and reconciliation out of React components.
- Treat names as display text, never durable identity.
- Do not hand-edit generated catalog modules as a substitute for the acquisition/review/generation
  process.
- A push to `master` triggers production deployment. Never push or merge `master` without explicit
  user authorization.

`src/tests/aos4/architectureBoundaries.test.ts` enforces the domain dependency boundary.

## Product and interface continuity

The live application at `https://aosreminders.com/` is the visual and interaction baseline. The
community trusts that experience; data, dependency, and framework work does not authorize a
redesign.

- Preserve the established dark-blue masthead, typography, spacing, edit/play control, faction
  selector, teal selection cards, reminder cards, notes, hide/show behavior, responsive layout,
  print behavior, footer, and contact/disclaimer treatment.
- Preserve the account-facing shell: the signed-out `Subscribe`, `FAQ`, and `Log in` navigation;
  Auth0 hosted login; authenticated `Profile` and `Log out` navigation; subscription status and
  cancellation; and subscriber theme behavior.
- Compare UI changes directly against the live site at desktop and mobile widths before accepting
  them. Browser snapshots and tests should guard recognizable landmarks and account navigation.
- Treat any UI change the user did not explicitly ask for as a code smell. Structural, dependency,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daviseford/aos-reminders](https://github.com/daviseford/aos-reminders) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
