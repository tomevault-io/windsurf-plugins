---
trigger: always_on
description: Expert in design tokens using the DTCG specification. Use this skill when users ask about design tokens, DTCG format, token validation, formatting, transformation, color spaces (sRGB, Display P3, OKLCH), references and aliasing, resolvers, theming with modifiers/contexts, multi-platform design systems, accessibility, or working with tools like jq, jsonata, and terrazzo. Helps with token file creation, resolver configuration, structure, naming conventions, and best practices.
---


# Design Tokens Expert

Expert guidance for working with design tokens following the Design Tokens Community Group (DTCG) specification.

## Quick Reference

| Topic | Reference |
|-------|-----------|
| Token types, structure, validation | [reference/format.md](reference/format.md) |
| Color spaces, components, alpha, **common mistakes** | [reference/color.md](reference/color.md) |
| Sets, modifiers, resolution order | [reference/resolver.md](reference/resolver.md) |
| jq, JSONata, **Figma export**, **Terrazzo config** | [reference/tools.md](reference/tools.md) |
| Common patterns and examples | [examples/use-cases.md](examples/use-cases.md) |

**Getting Started:** See [Getting Started Guides](#getting-started-guides) for step-by-step workflows.

## Specification Sources

Based on the latest DTCG Draft Community Group Reports:
- [Format Module](https://www.designtokens.org/tr/drafts/format/)
- [Color Module](https://www.designtokens.org/tr/drafts/color/)
- [Resolver Module](https://www.designtokens.org/tr/drafts/resolver/)

## Core Concepts

### Token Structure

A token is a JSON object with `$value`. Special properties use `$` prefix:

```json
{
  "brand-blue": {
    "$type": "color",
    "$value": {
      "colorSpace": "srgb",
      "components": [0.15, 0.39, 0.92],
      "hex": "#2563eb"
    },
    "$description": "Primary brand color"
  }
}
```

### Token Types

**Atomic:** color, dimension, fontFamily, fontWeight, duration, cubicBezier, number

**Composite:** strokeStyle, border, shadow, gradient, typography, transition

See [reference/format.md](reference/format.md) for complete type definitions.

### Color Format

Colors use structured objects (not hex strings):

```json
{
  "$type": "color",
  "$value": {
    "colorSpace": "srgb",
    "components": [1, 0, 0.5],
    "alpha": 0.8,
    "hex": "#ff0080"
  }
}
```

Supported spaces: srgb, display-p3, oklch, oklab, hsl, hwb, lab, lch, and more. See [reference/color.md](reference/color.md).

### References (Aliasing)

**Two syntaxes supported:**

1. **Curly braces** - Token-level references: `"{path.to.token}"`
2. **JSON Pointer (`$ref`)** - Property-level access: `{"$ref": "#/path/to/$value/property"}`

Token reference example:
```json
{
  "color": {
    "primary": {"$type": "color", "$value": {"colorSpace": "srgb", "components": [0, 0.4, 0.8]}}
  },
  "button": {
    "background": {"$type": "color", "$value": "{color.primary}"}
  }
}
```

JSON Pointer example (accessing array elements):
```json
{
  "blue": {
    "$type": "color",
    "$value": {"colorSpace": "okhsl", "components": [0.733, 0.8, 0.5]}
  },
  "blue-hue": {
    "$type": "number",
    "$ref": "#/blue/$value/components/0"
  }
}
```

See [reference/format.md](reference/format.md) for complete reference syntax details.

### Groups and Type Inheritance

Groups organize tokens. `$type` on a group applies to all children:

```json
{
  "spacing": {
    "$type": "dimension",
    "sm": {"$value": {"value": 8, "unit": "px"}},
    "md": {"$value": {"value": 16, "unit": "px"}},
    "lg": {"$value": {"value": 24, "unit": "px"}}
  }
}
```

### Resolvers for Theming

Resolvers manage tokens across contexts (themes, platforms, densities):

```json
{
  "name": "my-system",
  "version": "2025.10",
  "sets": {
    "core": {"sources": [{"$ref": "tokens/base.json"}]}
  },
  "modifiers": {
    "theme": {
      "contexts": {
        "light": [{"$ref": "themes/light.json"}],
        "dark": [{"$ref": "themes/dark.json"}]
      },
      "default": "light"
    }
  },
  "resolutionOrder": [
    {"$ref": "#/sets/core"},
    {"$ref": "#/modifiers/theme"}
  ]
}
```

See [reference/resolver.md](reference/resolver.md) for complete documentation.

## Getting Started Guides

### Quick Start: Convert CSS Variables to DTCG

**Starting point:** Existing CSS custom properties
```css
:root {
  --color-primary: #2563eb;
  --color-background: #ffffff;
  --spacing-sm: 8px;
  --spacing-md: 16px;
}
```

**Step 1:** Create primitives.tokens.json
```json
{
  "color": {
    "primitive": {
      "$type": "color",
      "blue-500": {
        "$value": { "colorSpace": "srgb", "components": [0.145, 0.388, 0.922], "hex": "#2563eb" }
      },
      "white": {
        "$value": { "colorSpace": "srgb", "components": [1, 1, 1], "hex": "#ffffff" }
      }
    }
  },
  "spacing": {
    "$type": "dimension",
    "scale": {
      "sm": { "$value": { "value": 8, "unit": "px" } },
      "md": { "$value": { "value": 16, "unit": "px" } }
    }
  }
}
```

**Step 2:** Create semantic.tokens.json with references
```json
{
  "color": {
    "interactive": {
      "$type": "color",
      "primary": { "$value": "{color.primitive.blue-500}" }
    },
    "background": {
      "$type": "color",
      "page": { "$value": "{color.primitive.white}" }
    }
  },
  "spacing": {
    "$type": "dimension",
    "component": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilikescience/design-tokens-skill](https://github.com/ilikescience/design-tokens-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
