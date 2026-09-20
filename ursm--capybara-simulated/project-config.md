---
trigger: always_on
description: This driver runs Capybara tests in-process: a V8-resident DOM (lives in
---

# capybara-simulated — engineering principles

This driver runs Capybara tests in-process: a V8-resident DOM (lives in
`lib/capybara/simulated/js/bridge.js`) driven through rusty_racer, with Nokogiri reserved
for the Rack response side. The codebase has a few load-bearing rules;
deviations have repeatedly cost us regressions or paint us into a
corner.

## 1. Spec conformance is the bar; real-browser behavior is how we check it

This driver exists to run real app suites in-process, so it has to
behave like a real browser. The **primary, objective correctness bar is
spec conformance**, measured by the vendored web-platform-tests gate
(`spec/wpt_gate/`, defined in `spec/support/wpt_gate.rb` — the same tests
Chromium / Firefox hold themselves to). Fix the spec contract and every
library built on it works for free.

The app suites (Avo / Discourse / Forem / Redmine / Mastodon) are the
**integration check and regression early-warning** — they catch
real-world breakage WPT can't: library interaction, ordering across many
APIs, the actual workflows the driver exists for. But they are **not a
frozen-behavior contract**. Keeping every existing app test green with no
changes is *not* a goal. When spec conformance conflicts with a behavior
an app test happened to rely on, favor the spec: make the driver
spec-correct and update the test. Do **not** grow a driver hack to
preserve a quirk (that's rule 2), and don't spend effort chasing
driver-dependent edge cases just to lift a green count.

This is what lets us make foundational pieces more spec-faithful even
when it shifts app-test timing — e.g. moving the timer / event-loop model
from the pragmatic wall-sync clock toward a real HTML event loop (task
queues + microtask checkpoints + spec timer ordering).

### In scope vs out of scope

Out-of-scope status is **earned by showing why a subtest can't be
satisfied**, never assumed because a fix looks like work. The default is
**in scope**. A subtest is out of scope (allowlisted / skipped, *not* a
driver bug) only when one of these holds:

1. **It needs a subsystem we deliberately don't model.** A *rendering*
   engine — glyph SHAPING (kerning, ligatures, bidi, the line-BREAKING
   algorithm), `display: contents` — a real async
   runtime, or legacy-multibyte / Unicode-version-tied
   encoding tables (ISO-2022-JP & friends; the *residual* IDNA cases where
   `uri-idna` diverges from the WPT reference — **not IDNA wholesale**: see
   the in-scope note below).
2. **It's a spec edge no real browser-built library or app depends on,
   AND satisfying it would require a library-shaped hack (rule 2) or a
   *measured* performance regression (rule 3).** Examples: attribute /
   property names around the 2³² index boundary; `Object.freeze` on a
   platform exotic object.

Everything else is in scope — fix it, favouring spec over app-quirk.
Cost and risk decide **priority and approach (incremental, perf-safe,
validated), not whether.** A high-cost-but-correct change (e.g. the
namespaced-attribute model: SVG `xlink:href`, case-sensitivity — all
real contracts) is scheduled as a careful staged effort, never skipped
for being tedious. "Addressable but annoying" is a backlog item, not an
exclusion. **"Not modeled yet / haven't built it" is never itself a reason
— that's the backlog.** An earned out-of-scope names the *subsystem* and
*why it can't be satisfied here*, not the effort.

**Already in scope — do NOT re-exclude these** (each has been wrongly
earned-out before as "a subsystem we don't model", then reverted):
- **Multi-origin: cross-origin iframes, SOP, postMessage-origin,
  `document.domain`, storage / Blob-URL partitioning.** Buildable in-process
  plumbing on parts we already have (per-frame V8 realms, the Rack harness);
  cross-origin is pure ORIGIN TAGGING, not a network boundary — no real DNS
  / `*.localhost` needed. Backlog, not a non-goal. (See the
  `multi-origin-in-scope` memory.)
- **Box layout is MODELED** (`js/src/layout.js`, since v0.8.0): block flow,
  inline runs, absolute / relative / fixed (shrink-to-fit included), margin
  collapsing, floats, FLEX layout (line breaking + grow/shrink distribution),
  a coarse grid pass, CSS Tables 3 auto/fixed TABLE layout, overflow
  clipping, the flat tree, cross-realm frames — and the page-visible geometry
  (`getBoundingClientRect` / `elementFromPoint` / `offset*` / `client*` /
  `scroll*`) reads from it, so there is ONE geometry. Visual hit-testing,
  gBCR truthiness and viewport-clip visibility are therefore IN scope: a
  failing geometry test is a coarse-model gap to diagnose (does it need glyph
  SHAPING, or just a box rule we haven't written?), not an
  automatic "needs a layout engine" exclusion.
- **FLEX SIZING is IN SCOPE** — the "flex / grid track sizing" clause above was
  written before `layout.js` existed and was retired 2026-08-31. `flexLines`
  breaks lines and `resolveFlexRowWidths` distributes free space over
  `flex-grow` / `flex-shrink` today; the css-flexbox allowlist earns out exactly
  six `.tentative` files and nothing for sizing, and the ~1000 remaining
  subtests are coarse-model gaps (e.g. the scrollable overflow region), not a
  missing algorithm. css-grid is simply not vendored — un-measured, not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ursm/capybara-simulated](https://github.com/ursm/capybara-simulated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
