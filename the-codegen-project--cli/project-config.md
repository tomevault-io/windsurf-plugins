---
trigger: always_on
description: time description: Modelina presets development guidelines for AsyncAPI Modelina integration
---

time description: Modelina presets development guidelines for AsyncAPI Modelina integration
globs: **/*
alwaysApply: true
---

## Modelina Presets Development Guidelines

### Overview
Modelina uses **presets** as middleware layers to customize generated models. Presets work as stackable layers that can extend, modify, or completely overwrite existing generated code.

### Preset Structure and Patterns

#### Basic Preset Structure
```typescript
// Basic preset object structure
{
  [modelType]: {
    [method]: (args) => { /* customization logic */ }
  }
}

// Example: TypeScript class preset
{
  class: {
    property({ content, model, property }) {
      return `// Custom comment\n${content}`;
    },
    additionalContent({ content, model, renderer }) {
      return `${content}\n${customMethods}`;
    }
  }
}
```

#### Preset with Options (Reusable Presets)
```typescript
// Reusable preset with options
{
  preset: {
    class: {
      property({ content, options }) {
        const description = options?.description || '// Default comment';
        return `${description}\n${content}`;
      }
    }
  },
  options: {
    description: '// Custom description'
  }
}
```

### TypeScript Preset Methods

#### Class Presets (`ConstrainedObjectModel`)
- `self` - Customize entire class rendering
- `ctor` - Extend constructor
- `property` - Customize property rendering
- `getter` - Extend property getters
- `setter` - Extend property setters
- `additionalContent` - Add methods/content to class

#### Interface Presets (`ConstrainedObjectModel`)
- `self` - Customize entire interface
- `property` - Customize property rendering
- `additionalContent` - Add additional interface content

#### Enum Presets (`ConstrainedEnumModel`)
- `self` - Customize entire enum
- `item` - Customize individual enum items

#### Type Presets (`ConstrainedUnionModel`, etc.)
- `self` - Customize type alias rendering

### Common Preset Patterns in Codebase

#### Using Built-in Presets
```typescript
import { TS_COMMON_PRESET, TS_DESCRIPTION_PRESET } from '@asyncapi/modelina';

const generator = new TypeScriptFileGenerator({
  presets: [
    TS_DESCRIPTION_PRESET,
    {
      preset: TS_COMMON_PRESET,
      options: {
        marshalling: true
      }
    }
  ]
});
```

#### Custom Validation Preset
```typescript
{
  class: {
    additionalContent: ({ content, model, renderer }) => {
      if (!generator.includeValidation) {
        return content;
      }
      return `${content}\n${generateTypescriptValidationCode({ model, renderer })}`;
    }
  }
}
```

#### Union Type Marshalling Preset
```typescript
{
  type: {
    self({ model, content, renderer }) {
      if (model instanceof ConstrainedUnionModel) {
        return `${content}\n\n${renderUnionUnmarshal(model, renderer)}\n${renderUnionMarshal(model)}`;
      }
      return content;
    }
  }
}
```

### Best Practices

#### Preset Ordering
- **Order matters**: Presets are applied in array order
- Place foundational presets (like `TS_COMMON_PRESET`) first
- Add specific customizations after base presets
- Consider preset interactions when ordering

#### Content Manipulation
```typescript
// ✅ GOOD: Prepend content
property({ content }) {
  return `// Comment\n${content}`;
}

// ✅ GOOD: Append content  
property({ content }) {
  return `${content}\n// Comment`;
}

// ✅ GOOD: Complete override
property({ property }) {
  return `public ${property.propertyName}: ${property.type};`;
}

// ❌ AVOID: Modifying without preserving structure
property({ content }) {
  return content.replace(/private/g, 'public'); // Fragile
}
```

#### Model Type Checking
```typescript
// ✅ GOOD: Check model types before customization
self({ model, content, renderer }) {
  if (model instanceof ConstrainedUnionModel) {
    return customUnionLogic(model, content, renderer);
  }
  if (model instanceof ConstrainedObjectModel) {
    return customObjectLogic(model, content, renderer);
  }
  return content;
}
```

#### Options Handling
```typescript
// ✅ GOOD: Provide defaults for options
property({ content, options }) {
  const includeComments = options?.includeComments ?? true;
  const commentPrefix = options?.commentPrefix || '//';
  
  if (includeComments) {
    return `${commentPrefix} Generated property\n${content}`;
  }
  return content;
}
```

### Generator Configuration Patterns

#### Standard Generator Setup
```typescript
const modelinaGenerator = new TypeScriptFileGenerator({
  ...defaultCodegenTypescriptModelinaOptions,
  presets: [
    // Base presets first
    TS_DESCRIPTION_PRESET,
    {
      preset: TS_COMMON_PRESET,
      options: { marshalling: true }
    },
    // Custom presets after
    customValidationPreset,
    customSerializationPreset
  ],
  enumType: generator.enum,
  mapType: generator.map,
  rawPropertyNames: generator.rawPropertyNames,
  useJavascriptReservedKeywords: generator.useForJavaScript
});
```

### Preset Limitations and Conflicts

#### Known Limitations
- **Location Conflicts**: Multiple presets modifying the same location (e.g., before a property) will render sequentially, not merged
- **Rendering Order**: Later presets receive content from earlier presets, making order critical
- **No Coordination**: Presets cannot communicate with each other


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-codegen-project/cli](https://github.com/the-codegen-project/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
