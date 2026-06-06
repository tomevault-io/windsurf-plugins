---
trigger: always_on
description: Useful when typing schemas in json
---

# Zod v4 Best Practices

Enforce best practices for Zod v4 schema validation and type safety across the project.


## Rules

### zod-imports
Use consistent Zod v4 imports.

**Pattern:** `import.*zod`
**Suggestion:** Use `import { z } from 'zod/v4'` for explicit v4 imports or `import { z } from 'zod'` for main import (both work with v4). Avoid `import * as z from 'zod'`.

**Good Examples:**
```typescript
import { z } from 'zod/v4'
import { z } from 'zod'
import z from 'zod/v4'
```

**Bad Examples:**
```typescript
import * as z from 'zod'
```

### zod-schema-naming
Use consistent naming convention for Zod schemas.

**Pattern:** `const.*Schema.*=.*z\.`
**Suggestion:** Name schemas with 'z' prefix followed by descriptive name and 'Schema' suffix. Use PascalCase for the descriptive part.

**Good Examples:**
```typescript
const zUserSchema = z.object({...})
const zComponentPropertiesSchema = z.object({...})
const zCoralRootSchema = z.object({...})
```

**Bad Examples:**
```typescript
const userSchema = z.object({...})
const schema = z.object({...})
```

### zod-type-inference
Use z.infer for type extraction from schemas.

**Pattern:** `type.*=.*typeof.*Schema`
**Suggestion:** Use `z.infer<typeof schemaName>` instead of `typeof schemaName` for type extraction.

**Good Examples:**
```typescript
type UserType = z.infer<typeof zUserSchema>
export type CoralComponentPropertyType = z.infer<typeof zCoralComponentPropertySchema>
```

**Bad Examples:**
```typescript
type UserType = typeof zUserSchema
```

### zod-validation-methods
Use appropriate validation methods for different scenarios.

**Pattern:** `schema\.(parse|safeParse|parseAsync|safeParseAsync)`
**Suggestion:** Use `safeParse` for runtime validation with error handling, `parse` for throwing errors, `parseAsync` for async validation, and `safeParseAsync` for async validation with error handling.

**Good Examples:**
```typescript
const result = await zCoralRootSchema.safeParseAsync(data)
const validated = zUserSchema.parse(userData)
const result = zComponentSchema.safeParse(componentData)
```

**Bad Examples:**
```typescript
const result = zUserSchema.parse(userData) // without try/catch
```

### zod-descriptions
Add descriptions to complex schemas for better documentation.

**Pattern:** `z\.object\(\{[^}]*\}\)`
**Suggestion:** Use `.describe()` method to add descriptions to schema fields, especially for complex objects and enums.

**Good Examples:**
```typescript
name: zCoralNameSchema.describe('The name of the Coral Component')
type: z.enum(['COMPONENT', 'INSTANCE']).describe('The type of the Coral Component')
```

### zod-union-types
Use proper union types for multiple possible values.

**Pattern:** `z\.union\(\[.*\]\)`
**Suggestion:** Use `z.union([...])` for multiple possible types. For literal values, prefer `z.enum([...])` when possible.

**Good Examples:**
```typescript
z.union([z.string(), z.number()])
z.enum(['string', 'number', 'boolean'])
z.union([zCoralTSTypesSchema, z.array(zCoralTSTypesSchema).describe('An array of types')])
```

### zod-optional-nullish
Use appropriate optional/nullish handling.

**Pattern:** `z\.(optional|nullish)\(\)`
**Suggestion:** Use `.optional()` for truly optional fields (undefined), `.nullish()` for fields that can be null or undefined, and `.nullable()` for fields that can be null but not undefined.

**Good Examples:**
```typescript
description: z.string().optional()
defaultValue: z.any().nullish()
parentRef: z.string().nullable()
```

### zod-transform-usage
Use transforms for data conversion when needed.

**Pattern:** `z\.string\(\)\.transform`
**Suggestion:** Use `.transform()` for data conversion, especially for converting between formats (e.g., kebab-case to camelCase).

**Good Examples:**
```typescript
z.string().transform((key) => key.replace(/-./g, (match) => match.charAt(1).toUpperCase()))
```

### zod-record-schemas
Use proper record schemas for object validation.

**Pattern:** `z\.record\(.*\)`
**Suggestion:** Use `z.record(keySchema, valueSchema)` for validating objects with dynamic keys. Consider using `.describe()` for better documentation.

**Good Examples:**
```typescript
z.record(zCoralNameSchema, zCoralDesignTokenSchema).describe('The design tokens of the Coral Component')
z.record(z.string(), z.any()).nullish()
```

### zod-error-handling
Implement proper error handling for Zod validation.

**Pattern:** `safeParse.*\.success`
**Suggestion:** Always check `.success` property when using `safeParse` methods and handle both success and error cases appropriately.

**Good Examples:**
```typescript
const result = await zCoralRootSchema.safeParseAsync(html)
if (!result.success) {
  throw new Error(result.error.message)
} else {
  return result.data
}
```

### zod-schema-composition
Compose schemas using proper Zod methods.

**Pattern:** `z\.object\(.*\)\.(and|merge|extend)`
**Suggestion:** Use `.and()`, `.merge()`, or `.extend()` for schema composition. `.and()` for intersection, `.merge()` for union of objects, `.extend()` for adding fields.

**Good Examples:**
```typescript
zCoralNodeWithChildrenSchema.and(z.object({...}))
baseSchema.extend({ newField: z.string() })
```

### zod-literal-usage
Use z.literal for exact value matching.

**Pattern:** `z\.literal\(`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Coral-UI/core](https://github.com/Coral-UI/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
