---
trigger: always_on
description: **Chobble Template** is an Eleventy (11ty) v3.0.0 static site generator for small business websites with e-commerce capabilities. It uses **Bun** as the package manager and runtime.
---

# CLAUDE.md - AI Assistant Guide for Chobble Template

## Project Overview

**Chobble Template** is an Eleventy (11ty) v3.0.0 static site generator for small business websites with e-commerce capabilities. It uses **Bun** as the package manager and runtime.

### Key Features
- Content types: Products, Categories, Events, News, Menus, Locations, Properties, Reviews, Team profiles
- E-commerce: Shopping cart (LocalStorage), Stripe Checkout, PayPal, Quote/enquiry mode
- 10 pre-built themes with live theme editor
- Responsive images with LQIP (Low Quality Image Placeholders)
- SEO/Schema.org structured data, RSS feeds, iCal events

---

## Quick Reference

### Essential Commands
```bash
bun install          # Install dependencies (MUST use bun, not npm)
bun run build        # Build the site to _site/
bun run serve        # Development server with hot reload
bun test             # Full test suite (lint + build + tests + coverage)
bun run test:unit    # Unit tests only
bun run lint         # Check code with Biome
bun run lint:fix     # Auto-fix lint issues
bun run precommit    # Pre-commit checks
```

### Directory Structure
```
src/
├── _data/           # Site configuration (config.json, site.json, strings.json)
├── _includes/       # Reusable HTML components (~85 files)
├── _layouts/        # Page layout templates (~34 layouts)
├── _lib/            # Core JavaScript library
│   ├── build/       # JS bundling, SCSS, theme compilation
│   ├── collections/ # Eleventy collections (12 types)
│   ├── config/      # Configuration helpers
│   ├── eleventy/    # Eleventy plugins (~13 configs)
│   ├── filters/     # URL-based filtering for products/properties
│   ├── media/       # Image processing (sharp, eleventy-img)
│   ├── public/      # Frontend JavaScript (bundled by Bun)
│   └── utils/       # Pure utility functions
├── css/             # SCSS stylesheets
├── products/        # Product markdown files
├── categories/      # Category data
├── events/          # Event data
└── [content dirs]/  # news, menus, locations, properties, etc.

test/
├── unit/            # Unit tests by feature
├── integration/     # Integration tests
├── code-quality/    # Code quality checks
├── test-utils.js    # Shared test utilities & factories
└── TEST-QUALITY-CRITERIA.md  # Testing standards
```

---

## Import Aliases

Use Node.js subpath imports (defined in `package.json`):

```javascript
import { memoize } from "#utils/memoize.js";
import { configureProducts } from "#collections/products.js";
import { configureImages } from "#media/image.js";
import config from "#data/config.json" with { type: "json" };
import { ROOT_DIR } from "#lib/paths.js";
```

**Available aliases:**
| Alias | Path |
|-------|------|
| `#data/*` | `./src/_data/*` |
| `#lib/*` | `./src/_lib/*` |
| `#build/*` | `./src/_lib/build/*` |
| `#collections/*` | `./src/_lib/collections/*` |
| `#config/*` | `./src/_lib/config/*` |
| `#eleventy/*` | `./src/_lib/eleventy/*` |
| `#filters/*` | `./src/_lib/filters/*` |
| `#media/*` | `./src/_lib/media/*` |
| `#utils/*` | `./src/_lib/utils/*` |
| `#public/*` | `./src/_lib/public/*` |
| `#test/*` | `./test/*` |

---

## Code Conventions

### Eleventy Plugin Pattern
Files registering with Eleventy export a `configureX` function:

```javascript
export function configureProducts(eleventyConfig) {
  eleventyConfig.addCollection("products", ...);
  eleventyConfig.addFilter("getProductsByCategory", ...);
}
```

### Functional Programming Style
The codebase uses curried, composable functions extensively:

```javascript
// Use pipe() for function composition
import { pipe, filter, map, sort } from "#toolkit/fp/array.js";

pipe(
  filter(x => x > 0),
  map(x => x * 2),
  sort((a, b) => a - b)
)(numbers);

// Curried helpers are preferred
const isActive = filter(item => item.active);
const getName = map(item => item.name);
```

### Memoization Pattern
```javascript
import { memoize } from "#utils/memoize.js";

const expensiveComputation = memoize(
  async (input) => { /* ... */ },
  { cacheKey: (args) => JSON.stringify(args[0]) }
);
```

### Available Array Utilities (`#toolkit/fp/array.js`)
- `pipe(...fns)` - Left-to-right function composition
- `filter(predicate)`, `map(fn)`, `flatMap(fn)`, `reduce(fn, initial)` - Curried array methods
- `sort(comparator)` - Non-mutating sort
- `unique(arr)`, `uniqueBy(getKey)` - Deduplicate arrays
- `filterMap(predicate, transform)` - Filter and map in single pass
- `compact(arr)` - Remove falsy values
- `chunk(arr, size)` - Split into groups
- `pick(keys)` - Extract object properties
- `memberOf(values)`, `notMemberOf(values)` - Membership predicates
- `pluralize(singular, plural?)` - Format counts with pluralization
- `accumulate(fn)` - Safe array building in reduce

### Error Handling: Fail Fast, Never Mask

**Throw errors instead of returning fallback values.** When something unexpected happens, fail immediately with a clear error rather than disguising the problem with a default value.

```javascript
// BAD - masks the problem, makes debugging harder
const getProduct = (id) => products.find(p => p.id === id) ?? { title: "Unknown" };

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chobbledotcom/chobble-template](https://github.com/chobbledotcom/chobble-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
