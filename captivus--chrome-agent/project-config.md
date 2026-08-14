---
trigger: always_on
description: Drive a real Chrome through the Chrome DevTools Protocol (CDP), from the terminal, for AI agents.
---

# chrome-agent

Drive a real Chrome through the Chrome DevTools Protocol (CDP), from the terminal, for AI agents.

Install: `uv tool install chrome-agent` (or `pip install chrome-agent`). Requires Google Chrome or Chromium. One runtime dependency (`websockets`); no Playwright, no browser downloads.

## What it is

Address a Chrome **instance by name**, and send it **any CDP command** or **stream any CDP event**. Two channels:

- **One-shot** (`chrome-agent <inst> Domain.method '{json}'`) — act: send one command, print the result, disconnect (~70 ms).
- **Attach** (`chrome-agent attach <inst> +Event …`) — observe: hold a connection and stream events as JSON lines.

**The full protocol, tracked live.** chrome-agent forwards your `Domain.method` straight to Chrome — nothing is validated against a bundled schema. Any command, event, or domain your installed Chrome supports works, **including protocol surface newer than this build** (e.g. `CrashReportContext` isn't among the bundled bindings, yet `chrome-agent <inst> CrashReportContext.getEntries` returns a normal result over the CLI). So **`help <inst> [Domain[.method]]`, read live from the browser, is the authoritative, version-correct reference** — prefer it over any static list. The typed Python classes are a point-in-time snapshot, not a gate.

**`experimental` ≠ unstable.** Most of the live protocol is flagged experimental (domains carry the flag and their members inherit it), and it's tempting to avoid it — don't. Experimental items break at roughly the same rate as the stable core; what predicts churn is how actively a *domain* is developed, and CDP's busiest domains (Network, Runtime, Page, DOM) are stable-status. The one real experimental signal is **removal/rename**, and even that is rare. Practical posture: **use whatever capability you need regardless of flag; pin the Chrome version you test against; re-verify signatures via `help` on upgrade.**

## Operating a page: sense ⇄ act

An agent does two things in a loop: it **senses** the page and it **acts** on it. **Sensing is the default, continuous mode; acting is the intermittent intervention.** After you act you're sensing again — and that perception is both your confirmation of the act *and* your orientation for the next one. There is no separate "verify" step; **the next sense is the verification.**

**Sense — two channels.** Match the channel to the question:

- **See what the page *says*** — `DOM`, `Accessibility`, `CSS`, `DOMSnapshot`. Structured reads are the primary, high-fidelity channel for content/structure/state. Use a **screenshot** (`Page.captureScreenshot`) for what the page *looks like* — layout, an image, a CAPTCHA; it is the *last resort* for reading content (pixels are lossy, OCR-like).
- **Hear what the page *reports*** — `Network` (what it fetched: requests, responses, bodies), `Runtime` (console output, uncaught exceptions), `Log`, `Audits`.

**Act — trusted operation.** `Input` (mouse/keyboard/touch/gesture events Chrome marks **trusted**), `Page` (navigate, reload, dialogs), `Runtime` (`evaluate`, `callFunctionOn`).

**The discipline:** sense across **more than one channel**, and **never trust an act's return value — trust the next sense.** An action that "succeeded" with no error can still have done nothing; only an observed effect proves it. For an irreversible action (send/submit), the cleanest observed effect is the action surface *disappearing* from the DOM (the compose pane gone), not the button's return. Screenshot before asserting a page "is empty / requires login / is broken." Derive identifiers from data (DOM/state), not from pixels.

**Sense page-readiness before you act.** After `Page.navigate`, wait on an observable condition — `attach +Page.loadEventFired`, or poll `document.readyState === "complete"` (or the target element) via `Runtime.evaluate` in a short retry loop — not a fixed sleep; acting on a half-loaded page reads stale state. Minimal poll: `chrome-agent mysite-01 Runtime.evaluate '{"expression":"document.readyState","returnByValue":true}'` until it returns `"complete"`.

**Most work is sense-dominant.** Four ways to engage a page — only one of them acts:

- **Drive the UI** — locate → act → sense (below).
- **Read what it already loaded** — the DOM, and (when the framework exposes it) its in-memory state, which is more authoritative than the painted DOM.
- **Be the authenticated client** — `Runtime.evaluate` running `fetch()` *inside* the logged-in page inherits its session; call same-origin APIs with zero credential handling.
- **Observe** — `attach` and watch `Network`/`Page`/console events.

### Acting trustworthily

**Two ways to "click," not interchangeable.** A synthetic `element.click()` (via `Runtime.evaluate`) is fabricated in page JS. A trusted `Input.dispatchMouseEvent` enters Chrome's native pipeline at the compositor and reaches what synthetic clicks can't: cross-origin iframes, capture-phase-intercepted overlays, shadow-DOM overlays, UIs that gate on event trust. **Escalation rule:** when a synthetic click *silently no-ops*, escalate to real `Input` events — don't debug selectors or the DOM. Don't over-escalate; plain `click()` is fine on ordinary UIs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [captivus/chrome-agent](https://github.com/captivus/chrome-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
