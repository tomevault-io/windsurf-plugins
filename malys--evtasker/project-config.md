---
trigger: always_on
description: Rule-based automation for the SAIC MG4 and part of **EVSuite**. At ignition it evaluates
---

# AGENTS.md — EVTasker

Rule-based automation for the SAIC MG4 and part of **EVSuite**. At ignition it evaluates
rules and applies supported actions directly through the shared EVHardware safety layer.
EVProfile is optional and is used only for the "apply saved profile" action.

The workspace `AGENTS.md` and normative workspace `DESIGN.md` apply; this file contains
only automation-specific additions.

Commit author: malys.training@gmail.com

License exception: EVTasker's own sources use PolyForm Noncommercial 1.0.0, not the
workspace MIT default. EVHardware remains separately licensed.

## The one rule that shapes everything

**Automation never bypasses EVHardware.** EVTasker reads a snapshot and executes named,
typed catalogue actions through EVHardware; it never accepts or sends a raw property ID.
The low-level safety gate applies regardless of which app invokes the library. So:

- Vehicle access code, firmware routing and property/transaction IDs live in EVHardware,
  not in this app.
- Rule actions are serialized; no two automation writes may interleave.
- EVProfile IPC is limited to profile discovery/application and takes no raw property ID.

## EVProfile profile boundary

The EVProfile profile bridge is protected by a signature permission. Both apps must be
signed with the same key for profile discovery/application; all other Tasker actions work
without EVProfile. The Diagnostic tab reports profile-bridge and hardware-layer state
separately.

## Unreadable ≠ false

The whole engine rests on this. A condition whose value is missing from the snapshot is
`UNAVAILABLE`; a rule with an unavailable condition is *not evaluable* and does not fire.
Never fill a missing reading with a default — that writes to a car on an assumption.
Covered by `ConditionEvaluatorTest` and `RuleEngineTest`.

## Firmware compatibility is annotation-driven

Every vehicle `ConditionType` / `ActionType` entry carries `@SupportedOn(...)`, derived
from EVHardware's `FirmwareInfo` and its per-generation routing. It is the single source
of truth for:

- `docs/firmware-matrix.md` — **generated** by `FirmwareMatrix`, refreshed by the test
  run. Never hand-edit it.
- the editor's runtime filter — entries unsupported on the connected car are hidden; an
  unknown firmware hides nothing.

Adding a vehicle entry without `@SupportedOn` fails `FirmwareSupportTest`.

## Triggers are events already received, not new listeners

`TaskerVehicleService` gets every ignition transition from one EVHardware listener. The
switch-off trigger reads the other end of that same stream — no second listener or bind.
Gear callbacks are not portable across the supported firmwares, so the P trigger samples
`EVHardware.isVehicleInPark()` every 500 ms only while ignition is RUN and fires only on a
confirmed non-P → P transition. Its first readable sample is a silent baseline, so service
recreation while already parked never fabricates an event. Physical buttons are conditions
(never a `RuleTrigger`): a rule containing one is
addressed by that event and excluded from vehicle-trigger cycles. EVHardware owns the OEM keycode
catalogue and short/long-press state machine; the app only receives the broadcast and feeds
its payload to that decoder. The receiver requires the
signature sender permission because that action is otherwise forgeable. Long press fires on
the OEM long event; its following release is suppressed instead of also firing short press.
Before adding a trigger, check whether something already delivers the event. Ignition repeats
are filtered in the service (`lastTrigger`), because the bus re-asserts states and a rule that
locks the doors must not run four times.

The manual test addresses only the selected rule (by id). It ignores that rule's trigger,
but must never evaluate the rest of the rule store.

`Rule.trigger` is **nullable on purpose**. Gson builds instances without calling the
constructor, so a Kotlin default never applies to a key absent from stored JSON; a non-null
enum would be null at runtime for every pre-existing rule. Read it through `firesOn`, export
the raw field. Same trap as the TypeToken bug — see `ShrinkerSafeGsonTest`.

## The standstill gate is fixed and fails closed

Vehicle-setting writes are allowed only when a valid speed read is exactly 0 km/h. Unknown,
negative or moving speed refuses the action; park state does not rescue an unreadable speed.
Do not expose a user-configurable moving threshold. Any existing `allowUpToKmh` or park-rescue
path is safety debt to remove, not a pattern to extend.

### The one carve-out: closing glass

The window actions (`SET_WINDOWS` and the four per-window writes) are gated **in the opening
direction only**. Closing a window while moving is allowed.

This is deliberate and is the sole exception. Closing the windows when it starts raining on
the motorway is the case those actions exist for, and a standstill gate would refuse exactly
that — the driver would be left with the rain coming in until the next red light. Opening
glass at speed is a vehicle-behaviour change and takes the gate like any other write.

The exception is carried by `ActionType.gatedWhenOpening`, not by `gated`, because it is a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malys/EVTasker](https://github.com/malys/EVTasker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
