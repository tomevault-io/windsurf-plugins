---
trigger: always_on
description: Native macOS Claude Code usage tracker. SwiftUI + SwiftData + Charts.
---

# Agent guide — Pacer

Native macOS Claude Code usage tracker. SwiftUI + SwiftData + Charts.
Single-binary menu-bar agent shape (LSUIElement=true). Two targets
(Pacer.app + PacerWidgets.appex) sharing data through an App Group.
See `docs/design.md` for the full v1 design.

## Where to look first

- `docs/design.md` — full architecture, data sources, schema, IPC, scope.
- `docs/research/ccusage-reference.md` — ground-truth analysis of `ccusage`
  internals: path discovery, JSONL schema, cost modes, dedup correctness,
  pricing source. Read before touching parsing or cost code.
- `docs/research/realtime-mechanisms.md` — analysis of statusline, hooks,
  OTel, MCP for live Claude Code data.
- `docs/research/tcc-app-management.md` — investigation of the
  every-launch "would like to access data from other apps" prompt,
  what was tried, current signing/notarization state, and the
  open SMAppService verification question for v1 release.
- `docs/research/ccusage-outputs/` — captured `bun x ccusage` JSON outputs
  for the local dataset. **Use these as ground-truth in tests** — every
  metric Pacer surfaces should match `ccusage`'s number for the same
  range, with exactly **two deliberate deviations**:
  1. the cache 5m/1h split (we track them separately, ccusage flattens);
  2. **output tokens on streamed messages** — ccusage dedups first-wins,
     which keeps the mid-stream snapshot instead of the finished message
     and under-counts output by ~63% on a real corpus (correctness rule
     §7). Pacer is deliberately higher here. If a ccusage comparison
     shows us reporting *more* output than ccusage, that is the fix
     working — do not "correct" it back.
- **`AGENTS.md` → "Performance — invariants and patterns"** (below) —
  read before adding ANY `@Query`, `FetchDescriptor`, computed view
  property, widget provider, or new rollup table. Codifies hard-won
  rules from five rounds of read-path optimization. The rules look
  nitpicky in isolation; in aggregate they're what keeps the app
  responsive while the in-process scan loop is firing every 5–60s.
- **`docs/perf-tuning.md`** — current cycle-time / CPU state, the
  measurement tooling (phase-timed scan log, `make perf-snapshot`),
  every perf commit's mechanism + measured win, and the open
  refactors that are deferred. Read before reintroducing animations,
  per-cycle SwiftData fetches, or adding any new always-running
  background work.

## Never take over the machine — no cursor, no windows, no focus

**Never without the owner's explicit, in-the-moment go-ahead.** Someone is
sitting at this Mac using it while you work. You do not get the input devices,
the windows, the focus, or the active Space — not briefly, not "just to check
something", not because a flag was set for you once in the past.

There is exactly one way this is allowed, and it is narrow:

1. You have already built and tested everything that *can* be tested off-screen,
   so the on-machine run is confirming one specific thing.
2. The whole run is scripted end to end, deterministic, and takes seconds. It
   asks nothing, guesses no coordinates, and restores what it touched.
3. You describe exactly what it will do, and the owner says go — **for that
   run**. Consent does not carry to the next one.

**Never explore, debug, or iterate on his screen.** If the scripted run fails,
it fails; take the artifacts away and work out why off-screen. A second attempt
needs a second go-ahead. "I'll just try it and see" is the thing that caused
this rule.

Concretely, never write, run, or leave behind anything that:

- posts synthetic input — `CGEvent`, `NSEvent` posting, `CGWarpMouseCursorPosition`,
  `CGDisplayMoveCursorToPoint`, `IOHIDPostEvent`;
- drives the UI through Accessibility or AppleScript — `osascript` with System
  Events, `AXUIElement` actions, `click`/`keystroke`/`key code`, `tell application
  … to activate`;
- runs an AppKit event loop against a *visible* window to simulate interaction —
  an `NSApp.run()` harness that dispatches mouse-moved events is exactly the
  thing this rule exists to stop;
- activates, raises, resizes, moves, closes or Spaces-switches any window,
  including Pacer's own;
- records the screen or captures another app's windows.

Reading is fine: `NSEvent.mouseLocation` to place a window the *user* asked
for, `NSScreen.frame`, and so on. The line is between observing the machine and
operating it.

**What to do instead.** Everything Pacer needs to see it can render off-screen,
headlessly, as a PNG — that is the entire reason `make render-live`,
`make screenshots` and `OffscreenRenderer` exist (next section). Behaviour that
is not visual belongs in a unit test. If something genuinely can only be
confirmed on a real session — an `NSMenu` tooltip is the standing example,
because NSMenu tracking cannot be exercised off-screen at all — then build it,
test your half headlessly, and **hand over a single prepared run**. Reporting
"this needs you to check" is a complete, acceptable answer.

Prefer making the *app* drive its own check over scripting coordinates from
outside: it knows where its own views are, so there is nothing to guess and
nothing to retry. See `PACER_TOOLTIP_SELFTEST` in `MenuBarTooltipSelfTest` for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EricAndrechek/Pacer](https://github.com/EricAndrechek/Pacer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
