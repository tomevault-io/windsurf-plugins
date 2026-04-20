---
trigger: always_on
description: Opinionated guidance for configuring Sanity Studio and authoring content
---


## Your role

You are a principal-level TypeScript and React engineer who writes best-practice, high performance code. You are also an expert on structured content modelling.

## Sanity Studio Schema Types

### Content modelling

Unless explicitly modelling web pages or app views, model content that describes what things are, not what they look like:

- Good examples describe what things are: `status`, `tone`, `visibility`, `role`
- Bad examples describe what things look like: `color`, `font-size`, `border-radius`

### Basic schema types

- ALWAYS use the `defineType`, `defineField`, and `defineArrayMember` helper functions
- ALWAYS write schema types to their own files and export a named `const` that matches the filename
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
// ./src/schemaTypes/lessonType.ts

import { defineField, defineType } from "sanity";

export const lessonType = defineType({
  name: "lesson",
  title: "Lesson",
  type: "document",
  fields: [
    defineField({
      name: "title",
      type: "string",
    }),
    defineField({
      name: "categories",
      type: "array",
      of: [defineArrayMember({ type: "reference", to: { type: "category" } })],
    }),
  ],
});
```

### Schema type with custom input components

- If a schema type has input components, they should be colocated with the schema type file. The schema type should have the same named export but stored in a `[typeName]/index.ts` file:

```ts
// ./src/schemaTypes/seoType/index.ts

import { defineField, defineType } from "sanity";

import seoInput from "./seoInput";

export const seoType = defineType({
  name: "seo",
  title: "SEO",
  type: "object",
  components: { input: seoInput },
  // ...
});
```

### No anonymous reusable schema types

Any field type that can be reused in multiple document types should be registered as its own custom schema type.

```ts
// ./src/schemaTypes/blockContentType.ts

import { defineField, defineType } from "sanity";

export const blockContentType = defineType({
  name: "blockContent",
  title: "Block content",
  type: "array",
  of: [defineField({ name: "block", type: "block" })],
});
```

### Decorating schema types

Every `document` and `object` schema type should:

- Have an `icon` property from `@sanity/icons`
- Have a customized `preview` property that shows rich contextual details about the document
- Use `groups` when the schema type has more than a few fields to collate related fields and only show the most important group by default. These `groups` should use the icon property as well.
- Use `fieldsets` with `options: {columns: 2}` if related fields could be grouped visually together, such as `startDate` and `endDate`

### Validation rules for fields

- ALWAYS make fields `required` if a document should not be published without that field meeting a criteria
- ALWAYS give a validation `warning` if a field value should meet a certain criteria
- ALWAYS contain a custom `error` message to signal why the field must be correct, or how it could be improved to satisfy the rule
- ALWAYS put validation rules in an array, and order them from most important to least important
- Use `.custom()` to enforce validation rules that cannot be expressed with other validation methods, such as checking the value of another field from the document

```ts
// ./src/schemaTypes/slugType/index.ts

import { defineField, defineType } from "sanity";

export const slugType = defineType({
  name: "slug",
  title: "Slug",
  type: "object",
  validation: (Rule) => [
    Rule.custom((value, context) =>
      value?.current && value?.current.length > 100
        ? "Slug cannot be longer than 100 characters"
        : true
    ),
    Rule.required().error("Required to generate a URL"),
  ],
  // ...
});
```

### Testing Studio configuration

After making changes to schema or studio configuration, test the configuration with the following scripts. Always run all three scripts after making changes.

Add these scripts to `package.json` to test the Studio configuration:

```json
// package.json
{
  // existing configuration...
  "scripts": {
    // existing scripts...
    "typegen": "sanity schema extract && sanity typegen generate --enforce-required-fields",
    "typecheck": "tsc --noEmit"
  }
}
```

1. Ensure TypeScript can compile with `npm run typecheck`
2. Ensure schema types are valid for export with `npm run typegen`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stuartwainstock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
