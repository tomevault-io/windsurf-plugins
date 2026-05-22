---
trigger: always_on
description: 1. [Foundational Principles for Salesforce UI Development](#foundational-principles-for-salesforce-ui-development)
---

## Table of Contents
1. [Foundational Principles for Salesforce UI Development](#foundational-principles-for-salesforce-ui-development)
2. [UX Principles](#ux-principles)
3. [Visual Design](#visual-design)
4. [Component Usage](#component-usage)
5. [Interaction Patterns](#interaction-patterns)
6. [Accessibility](#accessibility)
7. [Form Design](#form-design)
8. [Feedback and Validation](#feedback-and-validation)
9. [Layout and Responsiveness](#layout-and-responsiveness)
10. [Consistency and Reusability](#consistency-and-reusability)
11. [Theming and Customization](#theming-and-customization)
12. [Testing and QA](#testing-and-qa)
13. [Documentation and Onboarding](#documentation-and-onboarding)
14. [Feedback and Updates](#feedback-and-updates)

---

# Foundational Principles for Salesforce UI Development

Whenever developing Salesforce experiences—especially using LWC and SLDS—follow these foundational guidelines to ensure scalable, maintainable, and consistent UI code:

## Making Customizations: Decision Hierarchy

When writing UI code, always prioritize the following order for styling and layout decisions:

1. **Lightning Base Component/Variant**
   - Before writing any HTML, CSS, or JavaScript, determine if an existing Lightning Base Component (e.g., `lightning-button`, `lightning-card`) can achieve the desired appearance or behavior. For the full set of components and documentation, see the [Lightning Base Components Index](mdc:.cursor/rules/lightning-base-components/index.mdc).
   - Use a base component's out-of-the-box configuration options before adding any classes or writing custom CSS.

2. **Utility Class**
   - If the base component alone is insufficient, check if an SLDS utility class is available. Utility classes are especially important for composing "molecular" components—those made up of several smaller components that form part of a page or an entire page.
   - For the full set of utility classes, see [utilities](mdc:.cursor/rules/utilities/index.mdc).

3. **Custom CSS with Styling Token**
   - If neither a base component nor a utility class suffices, use the correct global styling hook (e.g., `--slds-g-color-surface-1`, `--slds-g-color-accent-1`) and assign it to the CSS property with a CSS variable. See [global-styling-hooks-color-usage.mdc](mdc:.cursor/rules/global-styling-hooks/global-styling-hooks-color-usage.mdc) for guidance on color usage, and only use real global styling hooks as specified in [slds-global-styling-hooks.mdc](mdc:.cursor/rules/global-styling-hooks/slds-global-styling-hooks.mdc).
   - Always use styling hooks in the correct semantic manner (never use a styling hook for an unintended purpose, such as `width: var(--slds-g-radius-border-circle)`).

> **Avoid all hard-coded values** except as a last resort (e.g., `height: 100%` if no utility class or styling hook exists).

---

## UX Principles
- The design should be easy to learn, understand, and navigate without explicit instruction.
- Respect progressive disclosure: don't overwhelm users.
- Use space, layouts, and data-density strategically to drive clarity.
- Use consistent interactions across flows, apps, and pages.
- Always test with assistive technologies, Salesforce customers, and internal users.

---

## Visual Design
> See [Foundational Principles for Salesforce UI Development](mdc:#foundational-principles-for-salesforce-ui-development) for the required order of styling and layout decisions.
- Follow SLDS spacing scale for margins and padding.
- Maintain brand-appropriate themes using semantic styling hooks.
- Establish information hierarchy using SLDS typography heading levels and card components.
- All text and interactive elements must meet WCAG 2.1 color contrast standards.
- All content must be in card containers.
- Text, icons, and other content elements must use appropriate spacing within cards. If no specific spacing is specified, use `slds-p-around_small`.
- Only use SLDS icons for any iconography requirements. See [Icons Index](mdc:.cursor/rules/icons/index.mdc) for the authoritative list of available icons and usage guidelines.
- Always use `<lightning-icon>` for SLDS icons.
- Standard Object icons should be used adjacent to record titles.
- Utility icons should be used for most other icon needs.
- **Never use Action icons.** (Action icons are reserved for platform use only and should not be used in custom apps or components.)

**Example: Card Container with Proper Spacing**
```html
<lightning-card class="slds-p-around_small">
  <div class="slds-text-heading_medium">Title</div>
</lightning-card>
```

**Example: Using a Styling Hook**
```css
.my-card {
  --slds-c-card-color-background: var(--slds-g-color-surface-1);
}
```

---

## Component Usage
> As outlined in the [Foundational Principles](#foundational-principles-for-salesforce-ui-development), always start with Lightning Base Components, then Utility Classes, then Styling Hooks.
- Use Lightning Base Components (e.g., `<lightning-card>`, `<lightning-datatable>`, `<lightning-button>`) as your first choice. Only write new LWC markup if a Lightning Base Component does not exist for the use case.
- If a Lightning Web Component needs customization, use styling hooks and slots.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [austin-guevara/lwc-slds-lbc-starter](https://github.com/austin-guevara/lwc-slds-lbc-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
