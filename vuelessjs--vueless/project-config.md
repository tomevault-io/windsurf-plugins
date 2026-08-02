---
trigger: always_on
description: Vueless UI is a Vue.js and Nuxt.js component library built with TypeScript and Tailwind CSS.
---

# Vueless UI Component Usage Examples for AI Agents

## Introduction
Vueless UI is a Vue.js and Nuxt.js component library built with TypeScript and Tailwind CSS.

This document provides comprehensive use case examples of Vueless UI components based on real-world usage patterns. 
Each example includes when to use the component, common props, events, and combinations.

Use `./node_modules/.cache/vueless/web-types.json` to get the actual and detailed specifications for each component's props, events, slots, and exposes.
Use `./node_modules/vueless` to in details understand Vueless UI components logic.

## Table of Contents

1. [Layout Components](#layout-components)
2. [Form Components](#form-components)
3. [Data Display Components](#data-display-components)
4. [Feedback Components](#feedback-components)
5. [Navigation Components](#navigation-components)
6. [Complex Component Combinations](#complex-component-combinations)

## Layout Components

Layout components form the structural foundation of your UI. 
They provide consistent spacing, alignment, and visual hierarchy while reducing the need for custom CSS or generic HTML elements.

## Layout Component Hierarchy

When building pages, follow this recommended hierarchy:

1. **UPage** - Top-level page container (for routed pages)
2. **UCard** - Content grouping and visual separation
3. **UGroups/UGroup** - Logical content sections with labels
4. **URow/UCol** - Flexible layout arrangement
5. **UDivider** - Content separation and visual breaks
6. **UModal/UModalConfirm** - Overlay containers for focused interactions
7. **UAccordion** - Collapsible content sections

## Best Practices

- **Start with structure**: Begin with layout components before adding content components
- **Minimize custom HTML**: Use Vueless layout components instead of generic `<div>` elements
- **Consistent spacing**: Leverage the built-in `gap` props rather than custom margins
- **Semantic grouping**: Use UGroup for labeled sections, UCard for visual grouping
- **Responsive design**: URow and UCol handle responsive behavior automatically
- **Progressive disclosure**: Use UAccordion for optional or secondary content

### UPage - Page Container

**When to use:** As the main container for top-level page content with navigation and actions. Use this component for:
- Full page views that correspond to application routes (e.g., `/users`, `/settings`, `/dashboard`)
- Pages that need a consistent header structure with title, back navigation, and action buttons
- Main content areas that require standardized spacing and layout

**When NOT to use:**
- Inside modal dialogs or nested components
- For partial page sections or reusable components
- In components that don't represent a complete page view

```vue
<UPage
    :back-to="backRouteParams"
    :back-label="t('title.settings')"
    :title="t('title.users')"
    size="xl"
    variant="soft"
    data-cy="users-page"
>
   <template #actions>
      <UButton :label="t('button.add')" @click="onAddUser" />
   </template>

   <!-- Page content -->
   <UTable :rows="tableItems" :columns="tableHeaders" />
</UPage>
```

### UCard - Content Card

**When to use:** For grouping related content into visually distinct sections. Use this component for:
- Form sections that need visual separation and optional headers/footers
- Content blocks that require consistent spacing and borders
- Dashboard widgets or information panels
- Any content that benefits from being contained within a card-like structure

**When NOT to use:**
- For simple content that doesn't need visual grouping
- When you need complex nested layouts (use URow/UCol instead)
- For full-page containers (use UPage instead)

```vue
<UCard
  :title="t('title.companyInfo')"
  :description="t('description.companyText')"
>
   <UCol>
      <UInput
          v-model="form.companyName"
          :label="t('label.companyName')"
          :error="companyNameError"
      />

     <UInput
       v-model="form.companyEmail"
       :label="t('label.companyEmail')"
       :error="companyEmailError"
     />
   </UCol>

   <template #footer-right>
      <UButton :label="t('button.continue')" @click="onSubmit" />
   </template>
</UCard>
```

### UGroups - Multiple Group Container

**When to use:** As a wrapper when you need to use multiple UGroup components in one page or component. Use this component for:
- Multi-section forms with multiple distinct groups that need proper spacing
- Settings pages with several configuration categories
- Complex layouts where multiple groups need consistent spacing between them
- Pages where groups should be visually separated with larger gaps

**When NOT to use:**
- When you only have a single UGroup (use UGroup directly)
- For simple content that doesn't need group-level organization
- When custom spacing between groups is needed (use manual spacing instead)

**Tip:**
- Use `upperlined` prop for all nested UGroup exclude of first.

```vue
<UGroups data-cy="user-settings-form">
   <UGroup :title="t('label.personalInfo')">
      <UCol>
         <UInput v-model="form.firstName" :label="t('label.firstName')" />
         <UInput v-model="form.lastName" :label="t('label.lastName')" />
         <UInput v-model="form.email" :label="t('label.email')" />

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vuelessjs/vueless](https://github.com/vuelessjs/vueless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
