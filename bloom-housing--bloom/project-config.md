---
trigger: always_on
description: Comprehensive web accessibility standards based on WCAG 2.2 AA, with 38+ anti-patterns, legal enforcement context (EAA, ADA Title II), WAI-ARIA patterns, and framework-specific fixes for modern web frameworks and libraries.
---


# Accessibility Standards

Comprehensive accessibility rules for web application development. Every anti-pattern includes a severity classification, detection method, WCAG 2.2 reference, and corrective code examples.

**Severity levels:**

- **CRITICAL** — Users cannot access content at all. Must be fixed before merge.
- **IMPORTANT** — Significant barrier for assistive technology users. Fix in same sprint.
- **SUGGESTION** — Improves usability for assistive technology. Plan for a future iteration.

---

## WCAG 2.2 Quick Reference (AA Level)

### Perceivable

| Criterion | Level | Summary |
|-----------|-------|---------|
| 1.1.1 Non-text Content | A | All non-text content has a text alternative. Decorative images use `alt=""`. |
| 1.2.1 Audio/Video-only | A | Provide transcript (audio) or text alternative (video). |
| 1.2.2 Captions (Prerecorded) | A | All prerecorded video has synchronized captions. |
| 1.3.1 Info and Relationships | A | Structure (headings, lists, tables, labels, landmarks) programmatically conveyed. |
| 1.3.2 Meaningful Sequence | A | When the sequence that content is presented affects its meaning, the visual and programmatic ordering of content should align. |
| 1.3.3 Sensory Characteristics | A | Instructions don't rely solely on shape, size, position, or sound. |
| 1.3.4 Orientation | AA | Content is not restricted to single orientation unless essential. |
| 1.3.5 Identify Input Purpose | AA | Input fields have `autocomplete` attributes when collecting information about the user. |
| 1.4.1 Use of Color | A | Color is not the only means of conveying info. |
| 1.4.3 Contrast (Minimum) | AA | Text: 4.5:1 normal, 3:1 large (18pt / 14pt bold). |
| 1.4.4 Resize Text | AA | Text resizable to 200% without loss of content or functionality. |
| 1.4.10 Reflow | AA | Sections of content can fit within 320px CSS width viewports without needing to scroll in two dimensions to read. |
| 1.4.11 Non-text Contrast | AA | UI components and graphics: 3:1 against adjacent colors. |
| 1.4.12 Text Spacing | AA | No loss of content or functionality with user-overridden line-height (1.5x), or specified paragraph spacing, letter spacing, and word spacing adjustments. |
| 1.4.13 Content on Hover/Focus | AA | Popup content that appears on hover or focus is: dismissible, hoverable, persistent. |

### Operable

| Criterion | Level | Summary |
|-----------|-------|---------|
| 2.1.1 Keyboard | A | All functionality operable via keyboard. |
| 2.1.2 No Keyboard Trap | A | User can navigate away from any component using keyboard. |
| 2.2.1 Timing Adjustable | A | Time limits can be extended or disabled. |
| 2.2.2 Pause, Stop, Hide | A | Auto-updating content can be paused. |
| 2.3.1 Three Flashes | A | No content flashes more than 3 times per second. |
| 2.4.1 Bypass Blocks | A | Skip link to bypass repeated navigation. |
| 2.4.2 Page Titled | A | Pages have descriptive `<title>`. |
| 2.4.3 Focus Order | A | Focus order preserves meaning and operability. |
| 2.4.4 Link Purpose | A | Link purpose determinable from text or context. |
| 2.4.6 Headings and Labels | AA | Headings and labels describe topic or purpose. |
| 2.4.7 Focus Visible | AA | Keyboard focus indicator is visible. |
| 2.4.11 Focus Not Obscured | AA | Focused element not entirely hidden by other overlaying elements (such as sticky headers or footers). *(New in 2.2)* |
| 2.5.1 Pointer Gestures | A | Multi-point gestures have single-pointer alternative. |
| 2.5.2 Pointer Cancellation | A | Activation on up-event, unless activation can be aborted, reversed, or down-event activation is essential. |
| 2.5.3 Label in Name | A | Accessible name contains the label text as it is visually presented. |
| 2.5.4 Motion Actuation | A | Device motion has UI alternative and can be disabled. |
| 2.5.7 Dragging Movements | AA | Drag-and-drop has click/tap alternative. *(New in 2.2)* |
| 2.5.8 Target Size (Minimum) | AA | Interactive controls have a target size, or spacing of at least 24x24 CSS px. *(New in 2.2)* |

### Understandable

| Criterion | Level | Summary |
|-----------|-------|---------|
| 3.1.1 Language of Page | A | `<html lang="...">` set correctly. |
| 3.1.2 Language of Parts | AA | Content in different language marked with `lang` attribute. |
| 3.2.1 On Focus | A | Focus doesn't trigger unexpected context change. |
| 3.2.2 On Input | A | Changing input doesn't auto-trigger unexpected context change. |
| 3.2.6 Consistent Help | A | Help mechanisms in same relative order across pages. *(New in 2.2)* |
| 3.3.1 Error Identification | A | Errors described to user in text. |
| 3.3.2 Labels or Instructions | A | Labels or instructions provided for user input. |
| 3.3.3 Error Suggestion | AA | Suggest corrections for detected errors. |
| 3.3.4 Error Prevention | AA | Submissions are reversible, checked, or confirmed. |
| 3.3.7 Redundant Entry | A | Don't re-ask for info already provided in same process. *(New in 2.2)* |
| 3.3.8 Accessible Authentication (Minimum) | AA | No cognitive function test (puzzle CAPTCHA). Allow paste and autofill. *(New in 2.2)* |

### Robust


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bloom-housing/bloom](https://github.com/bloom-housing/bloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
