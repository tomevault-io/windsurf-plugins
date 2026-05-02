---
trigger: always_on
description: Intent reference: WCAG 2.2 for designers, screen reader design, keyboard navigation, cognitive and motor accessibility, inclusive design principles and testing methodology. Load when working on accessibility, a11y audits, inclusive design, or assistive technology.
---


# Accessibility Foundations

## WCAG 2.2 for Designers

The Web Content Accessibility Guidelines are organized around four principles: Perceivable, Operable, Understandable, Robust (POUR). Most designers encounter WCAG as a compliance checklist. That's the wrong frame. WCAG is a design specification — it tells you what your design must achieve for the full spectrum of human ability.

### Perceivable

Information and interface components must be presentable to users in ways they can perceive. This means: not everyone sees, not everyone hears, not everyone processes information the same way.

**Text alternatives (1.1):** Every non-text element that conveys information needs a text equivalent. Images need alt text. Icons need labels. Charts need data tables or summaries. Video needs captions and audio description. The question is always: if this visual element disappeared, would the user lose information?

**How designers get this wrong:** Decorative images with verbose alt text (screen readers will read "stock photo of a diverse team collaborating in a modern office" for every page — exhausting). Informative images with empty alt text. Complex charts with alt text that says "chart" instead of describing the data. Icons without any programmatic label. The fix is intentional alt text: describe what the image communicates, not what it depicts.

**Time-based media (1.2):** Video needs captions (synchronized text of spoken content) and audio description (narration of visual content). Audio needs transcripts. Live content needs real-time captions. This isn't just for deaf users — captions serve anyone in a noisy or quiet environment.

**Adaptable (1.3):** Information structure must be programmatic, not just visual. A heading that's bold and large but coded as a `<div>` is invisible to screen readers. A data table that's built with positioned divs instead of `<table>` elements loses its row/column relationships. Design decisions about hierarchy, grouping, and sequence must be implementable as semantic structure.

**Distinguishable (1.4):** Contrast ratios: 4.5:1 for normal text, 3:1 for large text (18px+ or 14px+ bold), 3:1 for UI components and graphics. These are minimums — aim higher for body text. Color must not be the only means of conveying information — a red error border needs an icon or text too, because not everyone perceives red. Text must be resizable to 200% without loss of content or functionality.

**New in 2.2 — Dragging movements (2.5.7):** Any functionality that uses dragging must have a non-dragging alternative. Drag-to-reorder must also offer move-up/move-down buttons or an alternative input method.

### Operable

Users must be able to operate the interface through multiple input methods — not just mouse and touch.

**Keyboard accessible (2.1):** Everything must work with a keyboard. Every interactive element must be focusable and activatable. No keyboard traps — users must be able to navigate away from any component. Custom keyboard shortcuts must not conflict with browser or assistive technology shortcuts.

**Enough time (2.2):** If content has a time limit, users must be able to turn off, adjust, or extend the limit. Session timeouts need warnings and extension options. Auto-updating content needs pause/stop controls. Moving or auto-playing content must be stoppable.

**Seizures and physical reactions (2.3):** No content that flashes more than three times per second. This isn't theoretical — flashing content can trigger seizures in people with photosensitive epilepsy. Motion animations should be reducible (respect prefers-reduced-motion).

**Navigable (2.4):** Provide a skip navigation link (first focusable element, links to main content). Use descriptive page titles. Focus order must be logical and predictable — typically matching visual reading order. Link text must make sense out of context ("Read more" fails this; "Read our accessibility policy" passes). Headings and labels must be descriptive.

**New in 2.2 — Focus not obscured (2.4.11):** When a component receives keyboard focus, it must not be entirely hidden by other content (sticky headers, modals, toasts). At least partially visible.

**New in 2.2 — Target size (2.5.8):** Interactive targets must be at least 24x24 CSS pixels, with certain exceptions (inline links, native browser controls). This benefits motor-impaired users, touch users, and everyone with large fingers on small screens.

### Understandable

Content and interface behavior must be understandable to the user.

**Readable (3.1):** Set the language of the page (lang attribute). Identify changes in language within the page. Define unusual words, abbreviations, and jargon. These seem like developer concerns, but they're design decisions — the designer decides what terminology to use and how to explain it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
