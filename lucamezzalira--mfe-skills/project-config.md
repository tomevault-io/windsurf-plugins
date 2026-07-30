---
trigger: always_on
description: Micro-frontend architecture: Micro-frontends decisions framework. Load when this topic is in scope; part of mfe-skills.
---


# Micro-frontends decisions framework

**Version**: 1.2 | **Skill**: understanding-mfe-architecture | **Source**: *Building Micro-Frontends* (O'Reilly)

The decisions framework is composed of four areas that must be resolved upfront because each one constrains the next:

1. **Define** — horizontal or vertical split?
2. **Compose** — client-side or server-side?
3. **Route** — client-side or server-side?
4. **Communicate** — event emitter, custom events, web storage, query strings?

These decisions are not independent — composition constrains routing, and split strategy constrains communication. The table at the end of this file shows the valid combinations.

---

## Decision 1 — Define: horizontal or vertical split?

**Vertical split**: one micro-frontend per view or group of views. Each team owns a complete business domain end-to-end — authentication, catalog, checkout. The micro-frontend owns all the routes under its domain.

- Closer to a traditional SPA or server-side rendering approach
- Each team has full autonomy — fewer cross-team coordination points
- Better fit for teams new to micro-frontends
- Recommended as the starting point for most organisations

**Horizontal split**: multiple micro-frontends composing the same view. Multiple teams share responsibility for parts of the same page.

- Requires more discipline and governance — without it, proliferation of micro-frontends creates a distributed monolith
- Higher granularity → higher coupling → context leakage risk
- Requires upfront investment in a solid development experience (local composition, testing across teams)
- Better fit for teams experienced with micro-frontends and with strong automation

**Both are valid and not mutually exclusive**. Some parts of an application may suit a vertical split, others a horizontal split. Start with vertical where possible and introduce horizontal where domain ownership genuinely requires it.

**The granularity warning**: resist the instinct to split finely. A large number of micro-frontends composing a single view is a warning sign — each additional micro-frontend increases the coordination surface in the container and risks pushing domain logic into the shell.

---

## Decision 2 — Compose: how does the shell assemble micro-frontends?

### Client-side composition

The application shell loads micro-frontends directly from a CDN at runtime, using a JavaScript or HTML entry point. The shell dynamically appends DOM nodes or initialises the JavaScript application.

**Works for**: both vertical and horizontal splits.

**Best fit**: highly interactive platforms where teams need maximum deployment independence and runtime flexibility — enterprise apps, desktop-like web platforms, video-on-demand. Teams should have strong frontend skills and be comfortable with distributed system complexity.

**Tools**: Module Federation, Single SPA, Native Federation, iframes.

**Trade-offs**:
- (+) Full team independence — remotes ship and the shell picks them up at next load
- (+) Lazy loading — micro-frontends only load when the user navigates to them
- (-) Runtime composition errors are possible (network failures, version mismatches)
- (-) More complex local development setup

### Server-side composition

The origin server assembles the page by retrieving micro-frontend fragments and stitching them together before sending HTML to the client.

**Works for**: primarily horizontal split.

**Best fit**: content-heavy, SEO-critical sites where first-paint performance and search indexability matter most — e-commerce, publishing, news. Requires careful scalability planning for burst traffic; personalised pages cannot rely heavily on CDN caching.

**Trade-offs**:
- (+) Full server-side rendering — content visible before JavaScript loads
- (+) Better SEO and performance for content pages
- (-) Scalability must be planned carefully — runtime server-side composition under burst traffic is non-trivial
- (-) Personalised pages cannot rely heavily on CDN caching

---

## Decision 3 — Route: how does the application direct users between views?

Routing is directly constrained by the composition choice.

**Client-side routing**: the application shell owns global routing logic — it retrieves the routing configuration and decides which micro-frontend to load. Each micro-frontend may have its own internal routing for sub-pages within its domain.

- Used with client-side composition (vertical split)
- Supports complex routing logic: authentication state, geo-localisation, feature flags
- The shell loads one micro-frontend at a time for vertical split; a container page manages multiple micro-frontends for horizontal split

### URL depth ownership (vertical split)

When using client-side composition with a vertical split, split routing by URL depth:

| Depth | Owner | Examples |
|-------|--------|----------|
| First path segment | Shell | `/`, `/catalog`, `/checkout` |
| Second segment onward | Micro-frontend | `/catalog/product/sku-1`, `/checkout/shipping` |

The shell answers: *which MFE is active for this top-level area?*  
The MFE answers: *what screen inside that area?*

**Implications:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucamezzalira/mfe-skills](https://github.com/lucamezzalira/mfe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
