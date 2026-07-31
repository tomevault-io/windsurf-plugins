---
trigger: always_on
description: description: Third-party integration guidelines (Sanity, Shopify, HubSpot, etc.)
---

---
description: Third-party integration guidelines (Sanity, Shopify, HubSpot, etc.)
globs: *.tsx, *.jsx, *.css, *.js, *.ts
---

# Third-Party Integration Guidelines

## Sanity CMS Integration

### Configuration & Setup
Use CDN for performance with stega for visual editing. Store credentials securely in environment variables. All Sanity files are organized in `/lib/integrations/sanity/` directory.

```typescript
// In lib/integrations/sanity/client.ts
export const client = createClient({
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET,
  apiVersion: '2024-03-15',
  useCdn: true, // Use CDN for better performance
  token: process.env.SANITY_API_WRITE_TOKEN, // Write token for editing
  stega: {
    studioUrl: process.env.NEXT_PUBLIC_SANITY_STUDIO_URL || '/studio',
  },
})
```

### Schema Management

#### Content Modelling
- Unless explicitly modelling web pages or app views, create content models for what things are, not what they look like in a front-end
- For example, consider the `status` of an element instead of its `color`

#### Basic Schema Types
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
import {defineField, defineType} from 'sanity'

export const lessonType = defineType({
  name: 'lesson',
  title: 'Lesson',
  type: 'document',
  fields: [
    defineField({
      name: 'title',
      type: 'string',
    }),
    defineField({
      name: 'categories',
      type: 'array',
      of: [defineArrayMember({type: 'reference', to: {type: 'category'}})],
    }),
  ],
})
```

#### Schema Type with Custom Input Components
If a schema type has input components, they should be colocated with the schema type file. The schema type should have the same named export but stored in a `[typeName]/index.ts` file:

```ts
// ./src/schemaTypes/seoType/index.ts
import {defineField, defineType} from 'sanity'
import seoInput from './seoInput'

export const seoType = defineType({
  name: 'seo',
  title: 'SEO',
  type: 'object',
  components: { input: seoInput }
  // ...
})
```

#### No Anonymous Reusable Schema Types
ANY schema type that benefits from being reused in multiple document types should be registered as its own custom schema type.

```ts
// ./src/schemaTypes/blockContentType.ts
import {defineField, defineType} from 'sanity'

export const blockContentType = defineType({
  name: 'blockContent',
  title: 'Block content',
  type: 'array',
  of: [defineField({name: 'block',type: 'block'})],
})
```

#### Decorating Schema Types
Every `document` and `object` schema type should:

- Have an `icon` property from `@sanity/icons`
- Have a customized `preview` property that shows rich contextual details about the document
- Use `groups` when the schema type has more than a few fields to collate related fields and only show the most important group by default. These `groups` should use the icon property as well.
- Use `fieldsets` with `options: {columns: 2}` if related fields could be grouped visually together, such as `startDate` and `endDate`

### Visual Editing
Always add `data-sanity` attributes for visual editing. Use SanityContextProvider for document access. Implement proper draft mode handling. Import from `/lib/integrations/sanity` directory.

```typescript
import { useSanityContext, RichText } from '~/lib/integrations/sanity'

export function MyComponent() {
  const { document } = useSanityContext()
  
  return (
    <div data-sanity={document._id}>
      <h1 data-sanity="title">{document.title}</h1>
      <div data-sanity="content">
        <RichText content={document.content} />
      </div>
    </div>
  )
}
```

### Data Fetching
Use proper perspective for draft vs published content. Implement caching strategies for performance. Handle errors gracefully with try-catch. Import from `/lib/integrations/sanity` directory. Use `~/utils/metadata` helpers for SEO optimization. All `sanityFetch` calls automatically use `cacheSignal()` for request cleanup.

```typescript
// In lib/integrations/sanity/queries/index.ts
import { sanityFetch } from '../fetch'
import { generateSanityMetadata } from '~/utils'

// Use sanityFetch (automatically includes cacheSignal)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkroomengineering/elastica](https://github.com/darkroomengineering/elastica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
