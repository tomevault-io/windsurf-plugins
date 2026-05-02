---
trigger: always_on
description: Rethinks experiences for different platforms and contexts — not just resizing, but reconceiving. Part of the Intent design strategy system. When an experience moves from desktop to mobile, web to TV, consumer app to kiosk, or visual interface to voice, the interaction model, information priority, and user context all change. Trigger when: adapting a design for a new platform, planning multi-device experiences, auditing cross-platform consistency, designing for TV/kiosk/voice/embedded, or when so
---



# Transpose — Adapt Across Contexts

## Overview

Responsive design is a layout concern. Transposition is a UX concern.

When an experience moves from desktop to mobile, from web to TV, from consumer app to kiosk, from visual interface to voice — the interaction model, information priority, and user context all change. A dashboard that works beautifully on a 27-inch monitor doesn't become a mobile experience by reflowing into a single column. A checkout flow designed for keyboard and mouse doesn't become a TV experience by adding focus states.

Transposition means rethinking the experience for each context, not shrinking it to fit. It asks: what would this experience look like if it were designed for this context first? What would we add? What would we never have included? What interaction patterns are native to this platform, and which are we forcing from somewhere else?

**When to activate this skill:** Moving a product to a new platform, planning multi-device strategy, auditing cross-platform UX, designing for non-standard contexts (TV, kiosk, voice, embedded), or anytime someone says "just make it responsive" and the problem is bigger than layout.

---

## Skill family

Transpose works alongside the full Intent skill system:

- **`/journey`**: Your flows need to work across the contexts transpose identifies. A checkout flow on desktop is not the same journey on mobile or voice — journey designs the sequence, transpose ensures it fits the context.
- **`/organize`**: Navigation structure may fundamentally change per context. Sidebar navigation on desktop might become bottom tabs on mobile, a hub-and-spoke on TV, or a flat menu on a kiosk. Organize provides the IA; transpose adapts it.
- **`/include`**: Cross-context design IS inclusive design. Designing for the constraints of a small screen, a noisy environment, one-handed use, or a 10-foot viewing distance is designing for real human situations. Include ensures accessibility; transpose ensures contextual fit.
- **`/fortify`**: Different contexts have different failure modes. Mobile loses connectivity. TV remotes run out of batteries. Kiosks get touched by greasy fingers. Fortify maps the failure modes; transpose maps the contexts where they occur.
- **`/blueprint`**: System architecture must support multi-context delivery. APIs need to serve different data shapes. State sync needs infrastructure. Blueprint maps the system; transpose defines what each context needs from it.
- **`/philosopher`**: A cross-cutting cognitive mode for sitting with transposition problems before jumping to solutions. Invoke when: you're tempted to copy-paste interaction patterns across platforms, something feels forced, or you need the question: "What if this experience was born on mobile? What would we never have added?"

---

## Core capabilities

### 1. Context analysis framework

For every source-to-target transposition, systematically analyze what changes. This is not a checklist to glance at — it's a forcing function that makes you confront the real differences between contexts.

**Input method:**
- Desktop: mouse + keyboard, precise targeting, hover states, right-click, keyboard shortcuts, drag-and-drop
- Mobile: touch, imprecise targeting (minimum 44px), gestures (swipe, pinch, long-press), no hover, virtual keyboard covers half the screen
- TV: D-pad remote, focus-based navigation, no direct pointing, limited text input, voice remote on some devices
- Voice: no pointing, no visual feedback loop, conversational turn-taking, confirmation through re-prompting
- Kiosk: touch-only (no hover, no keyboard unless on-screen), large targets, often gloved or wet hands
- Embedded/widget: constrained input matching host app, possibly no dedicated input at all

**Attention model:**
- Desktop: focused, multi-window, long sessions, user is "at work" (even for personal tasks)
- Mobile: fragmented, interruption-prone, multitasking, micro-sessions interspersed with life
- TV: lean-back, passive, shared screen, low cognitive effort tolerance
- Kiosk: goal-directed, time-pressured, public environment, zero learning curve expected
- Embedded: ambient, secondary to primary task, glanceable

**Screen real estate:**
Constraints AND opportunities. Small screens force focus — that's a feature, not a bug. Large screens enable overview and comparison — but also invite clutter. Analyze what each size makes possible, not just what it takes away.

**Connectivity:**
- Desktop/web: generally reliable broadband, but not always
- Mobile: spotty, variable speed, metered data in many markets
- TV: usually reliable home Wi-Fi, but smart TVs have weak processors
- Kiosk: dedicated connection, but outages happen in retail/public environments
- Embedded/IoT: intermittent, low-bandwidth, offline-first is often the right default

**Environment:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
