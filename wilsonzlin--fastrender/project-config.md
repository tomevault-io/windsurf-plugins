---
trigger: always_on
description: This file contains **repo-wide** rules shared by all workstreams.
---

# FastRender (agent instructions)

This file contains **repo-wide** rules shared by all workstreams.

## Workstreams

Pick one workstream and follow its specific doc. Work can proceed **in parallel** across all workstreams.

### Rendering Engine (correctness & capability)

- **Capability buildout (spec-first primitives)**: `instructions/capability_buildout.md`
- **Pageset page loop (fix pages one-by-one)**: `instructions/pageset_page_loop.md`

### Browser Application (user-facing product)

- **Browser chrome (tabs, navigation, address bar)**: `instructions/browser_chrome.md`
- **Browser responsiveness (performance, not aesthetics)**: `instructions/browser_responsiveness.md`
- **Browser page interaction (forms, focus, scrolling)**: `instructions/browser_interaction.md`

### JavaScript (full browser JS support)

- **JS engine (vm-js core, execution, GC)**: `instructions/js_engine.md`
- **JS DOM bindings (document, element, events)**: `instructions/js_dom.md`
- **JS Web APIs (fetch, URL, timers, storage)**: `instructions/js_web_apis.md`
- **JS HTML integration (script loading, modules, event loop)**: `instructions/js_html_integration.md`

### Quick reference: what each workstream owns

| Workstream | Owns | Does NOT own |
|------------|------|--------------|
| `capability_buildout` | CSS, layout algorithms, paint correctness | Page-specific fixes, browser UI |
| `pageset_page_loop` | Fixing specific pages end-to-end | Generic capability work |
| `browser_chrome` | Tabs, address bar, navigation, shortcuts | Visual design, page interaction |
| `browser_responsiveness` | Frame rate, input latency, scroll perf | Visual design, chrome functionality |
| `browser_interaction` | Forms, focus, selection, scrolling | Chrome UI, JS events |
| `js_engine` | vm-js execution, GC, spec compliance | DOM, Web APIs |
| `js_dom` | Document, Element, Node, events | Web APIs, script loading |
| `js_web_apis` | fetch, URL, timers, storage, crypto | DOM, engine internals |
| `js_html_integration` | Script loading, modules, event loop | DOM APIs, engine internals |

## Non-negotiables

- **No page-specific hacks** (no hostname/selector special-cases, no magic numbers for one site).
- **No deviating-spec behavior** as a "compat shortcut". Implement the spec behavior the page depends on (incomplete is OK; wrong is not).
- **No post-layout pixel nudging**; keep the pipeline staged (parse → style → box tree → layout → paint).
- **No panics** in production code. Return errors cleanly and bound work.
- **Keep Taffy vendored** (`vendor/taffy/`) and only use it for flex/grid; do not update it via Cargo.
- **JavaScript execution must be bounded**: the JS engine must support interrupts/timeouts and avoid unbounded host allocations.

## Philosophy & culture

**Read [`docs/philosophy.md`](docs/philosophy.md)** for hard-won lessons, mindset principles, and development wisdom.

**Read [`docs/triage.md`](docs/triage.md)** for priority order, failure classification, and the operating model.

Key points:
- **Correct pixels are the product.** Everything else exists to help us ship correct pixels faster.
- **90/10 rule**: 90% accuracy + capability, 10% performance + infra. This is the GOAL.
- **Data-driven method**: Inject, trace, collect, understand, systematize. This is the HOW.
- **Priority order**: Panics → Timeouts → Accuracy failures → Hotspots → Polish → Spec expansion.
- **No vanity work**: Changes that don't improve pageset accuracy, eliminate crashes, or reduce uncertainty for imminent fixes are not acceptable.
- **Ruthless triage**: If you can't turn a symptom into a task with a measurable outcome quickly, stop and split the work.

## What counts

A change counts if it lands at least one of:

- **New capability** (feature/algorithm implemented) with a regression.
- **Bugfix** (behavior corrected) with a regression.
- **Stability** (crash/panic eliminated) with a regression.
- **Termination** (timeout/loop eliminated) with a regression.
- **Conformance** (meaningful new WPT/fixture coverage).
- **UX improvement** (user-visible quality, responsiveness, visual polish).

### What does NOT count (guard against drift)

- **Tooling/infra-only work** unless immediately used to ship an accuracy/capability/stability win.
- **Perf-only work** unless it fixes a timeout/loop or makes an accuracy fix feasible.
- **Docs-only work** unless it removes confusion actively blocking fixes.

If you find yourself "improving the harness" without changing renderer behavior, **stop and implement the missing behavior**.

## System resources (RAM / time / disk) — mandatory safety

### The cardinal rule: assume everything can misbehave

FastRender processes hostile inputs (arbitrary web pages, user content, fuzzed data). **Any code path can go pathological**: infinite loops, exponential blowups, memory explosions, deadlocks, livelocks, signal-ignoring hangs.

This isn't paranoia — it's operational reality. A test with a subtle bug can spin forever. A layout algorithm can hit a degenerate case. A network request can hang on a broken server. Code under development is especially suspect.

**Every command you run must have hard external limits that cannot be bypassed by the code being run:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilsonzlin/fastrender](https://github.com/wilsonzlin/fastrender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
