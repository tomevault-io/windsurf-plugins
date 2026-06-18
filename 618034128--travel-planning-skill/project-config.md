---
trigger: always_on
description: >-
---


# Travel Planning Skill

Turn a loose travel wish ("I want to go to X, leave around 9, back by 6, I really
want to see Y, rest is up to you") into two deliverables:

1. A **route map** — an interactive map if a maps/places tool is available, otherwise shareable map links.
2. A **time-blocked daily schedule** — realistic, with travel legs, meals, and buffer time.

If the trip starts in another city, the inbound and return legs are part of the
plan too — query a real option (e.g. via a 12306 MCP) and recommend it, but never
buy the ticket yourself (see `references/transport-and-booking.md`).

The user can revise anything at any point ("drop that temple", "this is too
rushed", "add a coffee stop"), and you regenerate only what's affected.

## The one rule that defines this skill: confirm before you plan

Do **not** produce a route or a schedule until the user has confirmed the trip
parameters. This is not bureaucracy — itinerary planning is expensive to redo,
and a single wrong assumption (wrong day, wrong pace, a closed attraction)
cascades through every downstream stop. Confirming first is faster overall and
respects the user's time.

The user often says "everything else is up to you." That does **not** mean skip
confirmation — it means *you* fill the open slots with sensible defaults and then
show those defaults back for a quick yes/no. The user approves a plan-shape, not
a blank check. Surface your assumptions; let them veto.

Concretely, the flow is always: **Intake → Confirm → Build → Revise.** Never jump
from a partial request straight to a finished itinerary.

---

## Phase 1 — Intake & Confirmation

Goal: reach a confirmed, unambiguous trip spec while asking as few questions as
possible. Read `references/intake-checklist.md` for the full field list, the
default values to assume, and example confirmation summaries.

Steps:

1. **Extract** everything the user already gave. Re-read the whole conversation —
   they may have mentioned the city, dates, or must-sees earlier. Don't re-ask
   for anything already stated.

2. **Fill gaps with explicit defaults**, don't interrogate. For each unstated
   parameter, pick a reasonable default (see the checklist) instead of asking.
   Asking ten questions for a casual day trip is a worse experience than
   proposing defaults and letting the user correct one or two.

3. **Resolve the few things that genuinely change the plan.** Only these deserve
   a direct question, and only when you truly can't infer them:
   - dates / which day(s), if pacing or opening hours depend on it
   - daily start and end time (and where the day starts/ends — hotel? station?)
   - must-go places and any hard "won't go" exclusions
   - rough pace (packed vs relaxed) and transport mode (walk / transit / drive)
   - origin city + inbound/return timing, if the trip starts elsewhere — these legs
     are part of the plan (see `references/transport-and-booking.md`)
   Ask at most a small batch of these at once. Bundle them; don't drip one at a time.

4. **Present a compact confirmation summary** and explicitly wait. Format it so
   the user can approve in one glance or correct a single line. End with a clear
   prompt like "Look right? Tell me what to change, or say go and I'll build it."

   Example confirmation block:
   ```
   Here's what I've got — confirm or correct:
   • Destination: Nanjing (old city core)
   • When: this Saturday, one day
   • Hours: out from hotel near Xinjiekou ~9:00, back by ~18:00
   • Must-do: Ruins of St. Paul'... (your picks)
   • Pace: relaxed, ~4–5 stops + meals
   • Getting around: metro + walking
   • Assumed for you: lunch near midpoint, one coffee/dessert break, indoor backup if it rains
   Say "go" and I'll plot the route and schedule.
   ```

5. **Only after a clear "go"** (or equivalent confirmation) do you proceed to
   Phase 2. If the user corrects something, update the summary and reconfirm the
   changed part — you don't need to re-list everything.

A note on judgment: if the request is genuinely complete and unambiguous already
(all of: place, day, hours, must-sees, pace), you may present the summary and
build in the same turn — but still *show* the summary first so nothing is a
surprise. The gate is about no hidden assumptions, not about forcing a round trip.

---

## Phase 2 — Build the itinerary

Only enter this phase after confirmation. Produce both deliverables.

### Route map

Read `references/maps-and-routing.md` for the decision logic. In short:
- If a maps/places tool or MCP is available (e.g. a places search + map display),
  use it: look up each stop to get real coordinates, hours, and ratings, then
  render an ordered route.
- Otherwise, generate shareable map links with the helper:
  `python scripts/build_map_links.py` (see the script's `--help`). It builds
  Google Maps directions URLs for international trips and Amap/Baidu URLs for
  mainland-China trips, with waypoints in visiting order.
- For mainland China, default to Amap (高德) or Baidu, since Google Maps is
  unreliable there. For everywhere else, default to Google Maps.

### Inter-city transport (getting there & back)

Only when the trip starts in a different city. Read
`references/transport-and-booking.md`. In short:
- If a transport tool is available (e.g. a **12306 MCP** for mainland-China rail, or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [618034128/Travel-Planning-Skill](https://github.com/618034128/Travel-Planning-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
