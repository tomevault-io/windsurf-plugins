---
trigger: always_on
description: Turns a web app's interaction surface into a typed, traversable journey graph an LLM can
---

# hcifootprint — feature-work map

Turns a web app's interaction surface into a typed, traversable journey graph an LLM can
plan over. You author **actions** and name **journeys**; a **tool** is what is served to a
model. One authored sentence (`does`) is both the human's label and the model's tool
description. Everything relational — routes between pages, which action unblocks which — is
**derived from declarations made for other reasons**, never authored. **Trust the code**
where any doc disagrees.

**The vocabulary (1.10.0):** you declare the JourneyMap; the session is the Walker; the
recording carries both. `defineJourneyMap`/`JourneyMap` are permanent reference-equal aliases of
`buildNavigationGraph`/`NavigationGraph` (tree/appmap.ts) — both names ship forever, neither is a
rename, and there is deliberately NO `Walker` export: the walker is the session. Three movers move
it — **human** (a real click, `principal: 'user'`), **agent** (the four served verbs `whats_here` /
`why` / `do_action` / `did_it_work`), **guard** (your data: `when`/`enabledWhen`) — plus the world's
own `Cause.kind: 'stimulus'`. Same pattern at three altitudes: footprintjs walks stages,
agentfootprint (`defineSkillMap`) walks skills, this walks screens.

**An effect already held (1.12.0):** **An effect that is already true is not a pending one. When an
action's declarative verify contract covers every key it declares it writes and already holds at
fire time, the fire never waits for a state report that nothing will send — it settles on its own
handler and answers `alreadyTrue`.** The rule lives in ONE small module (traverse/already-true.ts,
five conditions) called from `fire()` before the record is minted; it flips exactly one boolean in
the state-tap arm's guard, so an already-true fire falls through to the tapless block and waits on
the HANDLER rail instead. It reads `verify` and NOT `writes` — `writes` is key names only by stated
law, so nothing here can know the value an action would set. The marker is `alreadyTrue:
FilterCondition[]` on the fire result AND the row (presence is the mark, the value is the evidence:
one field, not a boolean beside a list), served by modes.ts on the fire result and the settled
`did_it_work` answer with one authored sentence. NOT stamped on an allowed-unmaterialized fire —
that fate already has `materialized: false`.

**Refusals teach, including the four that did not (1.12.0):** `TRANSITION_ID_REQUIRED`,
`ACTION_REQUIRED`, `KEY_REQUIRED` and `JOURNEY_REQUIRED` were raised as `{ok:false, judgment:'error',
reason}` and nothing else — no `why`, no correction, no valid set, not even `positionData()`. They
now meet the house standard: the argument named, the correction attached, the valid set carried
where one exists (`actions` / `journeys` from the same doors `UNKNOWN_ACTION` / `UNKNOWN_JOURNEY`
read; `pending` + `awaitingSettlement` + `awaitingHuman` from the same doors `UNKNOWN_TRANSITION`
reads), and honest silence where none does (`KEY_REQUIRED` — `why` runs a slice and never refuses a
key, so there IS no valid set). ADDITIVE: the four `reason` strings are unchanged, none grows a
singular `transitionId` (consent-invariant.test.ts sweeps for that), and no `openTransitions` was
minted — `awaitingSettlement` is already the word for fires still open, and a second name for one
fact is what answer-grammar rule 3 forbids. One in-repo `toEqual` pin was loosened deliberately
(one-journey-tool.test.ts) — it was the only exact-match assertion on the four.

**Position has three tiers, one door each (1.11.0):** page (`sync`) → container
(`observeFocus`, new) → state (`updateState`, not position). **Sync pages; observe the deeper place.
`sync()` moves the walker and decides what is served; `observeFocus()` says which tab or area the
reader is in. Declare containers, and report the deepest one on screen.** The middle door had to
exist: actions are served from the PAGE, so a cursor on a tab is served nothing (a container path to
`sync` now syncs its page and warns, naming `observeFocus` — before 1.11.0 it went off-graph
silently), and `focus` moved only on `fire()`/`sync()`, so a PERSON clicking a tab could never move
it. `observeFocus` (traverse/nav-session.ts) sets `focus` + the new `lookingAt` getter (served beside
`youAreOn` by modes.ts `positionData`), records a `FocusMove`, and touches NOTHING else — no
transition, no version bump, no change to `available()`. It refuses BY NAME: undeclared node, or a
node on another page. Three facts, three doors: `show()`/`setVisible()` = VISIBLE,
`observeFocus()` = the READER, `sync()` = the WALKER. Authored half lintable
(`unevidenceable-tab`, advisory); the runtime half no static check can see.

## Before you design it: it may already exist

**Read this table before proposing any new capability.** Everything below already ships.
The failure it exists to stop is real and expensive: a reader searches for the words THEY
would use, finds nothing, and designs a feature this library has had for releases. It
happened three times in one day — twice here. Someone proposed "bind actions by role and
name instead of CSS selectors", and `ElementLocator` had been exactly `{ role, name }`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [footprintjs/hcifootprint](https://github.com/footprintjs/hcifootprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
