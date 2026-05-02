---
trigger: always_on
description: Intent reference: form design, state management, validation patterns, feedback loops, progressive disclosure, error recovery, undo/redo. Load when designing forms, interactions, input validation, or state transitions.
---


# Interaction Patterns

## Form Design Principles

Forms are where users exchange value with your product. Every unnecessary field, confusing label, or unhelpful error message is friction between the user and their goal. The research on form design is extensive and remarkably consistent.

### One Thing Per Page

The Government Digital Service (GDS) pattern, validated across millions of transactions: each screen asks one question or collects one piece of information. Not one form field — one conceptual unit.

**Why it works:** Reduces cognitive load. Each page has a clear purpose. Error recovery is simpler — the error is on this page, about this thing. Progress feels tangible. Mobile performance improves (less content per load). Analytics are more granular (you know exactly where drop-off occurs).

**When to bend it:** Closely related fields that users think of as one concept (first name + last name, city + state + zip). Editing flows where users need to see multiple fields to understand context. Expert tools where speed matters more than guidance.

**When not to bend it:** Checkout flows. Registration. Any flow where drop-off is a risk. Any flow used on mobile.

### Inline Validation Timing

When to validate is as important as how to validate. Get the timing wrong and validation becomes harassment.

**Validate on blur (leaving a field), not on keystroke.** Validating while the user is still typing is hostile — they haven't finished their input and you're already telling them it's wrong. Luke Wroblewski's research (2009) confirmed that inline validation improves completion rates, but only when triggered after the user leaves the field.

**Exception: password strength.** Real-time feedback on password requirements is one of the few cases where keystroke-level validation helps, because the user is building toward a goal and needs to know the criteria as they type.

**Exception: character counts.** If a field has a maximum length, show the remaining count as the user types. Don't wait until they've written a paragraph to tell them the limit is 140 characters.

**Validate once, then validate on change.** After a field shows an error, re-validate on every change so the error disappears the moment the input becomes valid. Don't make users re-submit the form to discover they've fixed the error.

### Field Grouping

Related fields should be visually and semantically grouped. Ungrouped forms feel longer than they are.

**Use fieldsets for conceptual groups:** Personal information, payment details, shipping address — each is a group. The `<fieldset>` element with a `<legend>` provides both visual grouping and accessibility structure.

**Limit visible fields.** If a form has 20 fields but only 6 are relevant based on previous answers, show 6. Conditional visibility reduces perceived complexity without hiding options. The user sees a short form that adapts, not a long form that wastes their time.

**Single-column layout.** Matteo Penzo's eye-tracking research (2006) and Baymard Institute's studies consistently show that single-column forms outperform multi-column forms. Users scan vertically; multi-column layouts create ambiguous reading order and increase completion time.

### Smart Defaults

Default values should serve the user's most likely intent, not the business's preferred outcome.

**Good defaults:** Country pre-filled from IP geolocation. Date pre-filled to today. Quantity defaulting to 1. Shipping address copied from billing address with one click.

**Bad defaults:** Pre-selected premium tier. Pre-checked marketing consent. Default to most expensive option. Privacy settings defaulting to "share with everyone."

**The test:** If 80% of users would choose this value, it's a good default. If the default primarily benefits the business, it's manipulation (see the anti-pattern catalog).

---

## State Machines for UI

Every interactive component exists in multiple states. Enumerating those states before building prevents the most common class of UI bugs: the states nobody designed for.

### Universal Component States

**Default/Resting** — The component as it appears before any interaction. This is what the user sees first. It must communicate: what is this, and what can I do with it?

**Hover** — Mouse cursor is over the component. Must communicate: this is interactive, something will happen if you click. Not applicable to touch interfaces — never put essential information in hover states.

**Focus** — Component has keyboard focus. Must be visually distinct from hover AND default. This is a hard accessibility requirement — users navigating by keyboard need to know where they are.

**Active/Pressed** — User is in the process of activating (mouse down, touch start). Must provide immediate tactile feedback that the action is registering.

**Disabled** — Component exists but can't be activated. Must communicate: this exists, but you can't use it right now (and ideally, why). Disabled states that provide no explanation for why they're disabled are a common frustration.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
