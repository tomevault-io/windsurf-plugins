---
trigger: always_on
description: description: Opinionated guidance for configuring Sanity Studio and authoring content with Schema UI Starter
---


---

description: Opinionated guidance for configuring Sanity Studio and authoring content with Schema UI Starter
globs: \*_/_.{ts,tsx,js,jsx}
alwaysApply: false

---

## Positive affirmation

You are a principal-level TypeScript and React engineer who writes best-practice, high performance code.

## Schema UI Architecture

### Component Structure

- The Schema UI Starter uses a composable block-based architecture
- Each block is defined as a schema type in Sanity and has a corresponding React component
- Blocks are organized in a hierarchical structure with parent blocks (like `split-row` or `grid-row`) containing child components

### Dynamic Component Mapping

- Each parent block component uses a `componentMap` object to map schema types to React components
- This pattern allows for type-safe rendering of dynamic content from Sanity

```ts
// Example component map pattern
const componentMap: {
  [K in ChildType["_type"]]: React.ComponentType<
    Extract<ChildType, { _type: K }>
  >;
} = {
  "child-type-1": ChildComponent1,
  "child-type-2": ChildComponent2,
};
```

## Sanity Studio Schema Types

### Content modelling

- Unless explicitly modelling web pages or app views, create content models for what things are, not what they look like in a front-end
- For example, consider the `status` of an element instead of its `color`
- Organize blocks into logical groups (hero, grid, split, carousel, etc.) for better content management

### Basic schema types

- ALWAYS use the `defineType`, `defineField`, and `defineArrayMember` helper functions
- ALWAYS write schema types to their own files and export a named `const` that matches the filename
- ALWAYS include an appropriate icon from `lucide-react` for each schema type
- ONLY use a `name` attribute in fields unless the `title` needs to be something other than a title-case version of the `name`
- ANY `string` field type with an `options.list` array with fewer than 5 options must use `options.layout: "radio"`
- ANY `image` field must include `options.hotspot: true`
- INCLUDE brief, useful `description` values if the intention of a field is not obvious
- INCLUDE `rule.warning()` for fields that would benefit from being a certain length
- INCLUDE brief, useful validation errors in `rule.required().error('<Message>')` that signal why the field must be correct before publishing is allowed
- AVOID `boolean` fields, write a `string` field with an `options.list` configuration
- NEVER write single `reference` type fields, always write an `array` of references
- CONSIDER the order of fields, from most important and relevant first, to least often used last

```ts
// ./sanity/schemas/blocks/grid/grid-row.ts

import { defineField, defineType } from "sanity";
import { LayoutGrid } from "lucide-react";
import { COLS_VARIANTS } from "../shared/layout-variants";

export default defineType({
  name: "grid-row",
  title: "Grid Row",
  type: "object",
  icon: LayoutGrid,
  fields: [
    defineField({
      name: "padding",
      type: "section-padding",
    }),
    defineField({
      name: "colorVariant",
      type: "color-variant",
      title: "Color Variant",
      description: "Select a background color variant",
    }),
    defineField({
      name: "gridColumns",
      type: "string",
      title: "Grid Columns",
      options: {
        list: COLS_VARIANTS.map(({ title, value }) => ({ title, value })),
        layout: "radio",
      },
      initialValue: "grid-cols-3",
    }),
    defineField({
      name: "columns",
      type: "array",
      of: [
        { type: "grid-card" },
        { type: "grid-post" },
        { type: "pricing-card" },
      ],
      options: {
        insertMenu: {
          views: [
            {
              name: "grid",
              previewImageUrl: (block) => `/sanity/preview/${block}.jpg`,
            },
            { name: "list" },
          ],
        },
      },
    }),
  ],
  preview: {
    select: {
      title: "columns.0.title",
      postTitle: "columns.0.post.title",
    },
    prepare({ title, postTitle }) {
      return {
        title: "Grid Row",
        subtitle: title || postTitle,
      };
    },
  },
});
```

### Parent-Child Block Structure

- Parent blocks (like `split-row` or `grid-row`) should contain an array field for child components
- Child components should be defined as separate schema types
- Use `validation: (rule) => rule.max(n)` to limit the number of child components when appropriate
- Use `options.insertMenu` to organize child components into logical groups with preview images

```ts
// Example parent block with child components
defineField({
  name: "splitColumns",
  type: "array",
  of: [
    { type: "split-content" },
    { type: "split-cards-list" },
    { type: "split-image" },
    { type: "split-info-list" },
  ],
  validation: (rule) => rule.max(2),
  options: {
    insertMenu: {
      views: [
        {
          name: "grid",
          previewImageUrl: (block) => `/sanity/preview/${block}.jpg`,
        },
        { name: "list" },
      ],
    },
  },
});
```

### React Component Implementation

- Create a corresponding React component for each schema type
- Use TypeScript to ensure type safety between schema and component

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serge-0v/next-js-sanity-starter](https://github.com/serge-0v/next-js-sanity-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
