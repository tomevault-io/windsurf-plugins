---
trigger: always_on
description: When adding a new helper to the `lo` library, follow these guidelines to create or update documentation files in the `docs/data/` directory.
---

# Adding New Helper Documentation

When adding a new helper to the `lo` library, follow these guidelines to create or update documentation files in the `docs/data/` directory.

## File Structure

Documentation files follow the naming pattern `core-{helper-name}.md` for core helpers.

### Frontmatter Format

Each documentation file must start with a frontmatter section:

```yaml
---
name: HelperName
slug: helpername
sourceRef: file.go#L123
category: core
subCategory: slice
signatures:
  - "func HelperName(params) returnType"
  - "func (receiver *Type) MethodName(params) returnType"
  - "func HelperNameI(params) returnType"
  - "func HelperNameWithContext(params) returnType"
playUrl: https://go.dev/play/p/EXAMPLE
variantHelpers:
  - core#slice#helpername
  - core#slice#helpername2
  - core#slice#helpername3
  - core#slice#helpername4
  - core#slice#helpername5
  - core#slice#helpernamei
  - core#slice#helpernamewithcontext
similarHelpers:
  - mutable#slice#helpername
  - core#slice#filterhelpername
  - core#slice#zipx
position: 0
---
```

### Frontmatter Fields

- **name**: The display name of the helper (PascalCase)
- **slug**: URL-friendly name (kebab-case, matches filename without `core-` prefix)
- **sourceRef**: Source file reference with line number (format: `file.go#L123`)
- **category**: `core`, `mutable`, `parallel`, `it`... The category must match the file name.
- **subCategory**: The functional category (e.g., `condition`, `map`, `find`, `slice`...)
- **signatures**: Array of function signatures as strings. Do not list signatures from other sub-packages/category.
- **playUrl**: Go Playground URL with working example
- **variantHelpers**: Array of variant helper names. Must contain at least the default helper named above. This field is for:
  - Variations of the same helper with different signatures (e.g., `Map`, `MapI`, `MapWithContext`, `MapIWithContext`)
  - Helper variants that add functionality like indexes, context, or different parameter types
  - All variants must be in the same category and subcategory as the main helper
  - Examples: `core#slice#map`, `core#slice#mapi`, `core#slice#mapwithcontext`
- **similarHelpers**: Array of related helper names (leave empty if none). This field is for:
  - Equivalent helpers in other packages/categories (e.g., `parallel#slice#Map`, `mutable#slice#Filter`)
  - Helper compositions or related functionality (e.g., `FilterMap` is similar to both `Map` and `Filter`)
  - Helpers with different names but similar purposes (e.g., `FindBy` variants vs base `Find`)
  - Cross-references to helpers that users might want to consider as alternatives
- **position**: Position in the list (0, 10, 20, 30...). Order must follow the order in source code. Helpers are grouped by category+sub-category and displayed on a page. Position number is reset for each page.

## Keeping sourceRef in Sync

The `sourceRef` field (format `file.go#L123`) points to a specific line number in the Go source. Any change to a `.go` file (adding, removing, or reordering functions) shifts line numbers for everything below the change in that file, which can make existing `sourceRef` values stale — not just for the function being edited, but for every other documented helper in the same file.

Whenever Go source code is modified:

1. Run `gopls symbols <file>` for every changed `.go` file to list all symbols (functions/methods/types) with their current line numbers.
2. Cross-reference each symbol against the `sourceRef` fields in `docs/data/*.md` files that document helpers from that file.
3. Update any `sourceRef` whose line number no longer matches.

## Content Structure

After the frontmatter, include:

1. **Brief description**: One sentence explaining what the helper does
2. **Code example**: Working Go code demonstrating usage
3. **Expected output**: Comment showing the result

```markdown
Brief description of what this helper does. Be concise and not too long.

```go
result := lo.HelperName(example)
// expected result
```
```

Multiple examples can be used for demonstration the helper, such as edge cases. If multiple signatures are grouped under this documentation, it could be useful to describe some (all?) of them.

## Understanding variantHelpers vs similarHelpers

### variantHelpers
Use `variantHelpers` for different versions of the **same helper function**:

**Example**: Map helper variants (all in `core#slice#map`):
```yaml
variantHelpers:
  - core#slice#map        # func Map[T, R]([]T, func(T, int) R) []R
  - core#slice#maperr     # func MapErr[T, R]([]T, func(T, int) (R, error)) ([]R, error)
  - core#slice#mapi       # func MapI[T, R]([]T, func(T, int) R) []R (with index)
  - core#slice#mapwithcontext  # func MapWithContext[T, R]([]T, func(T, int, context.Context) R, context.Context) []R
```

### similarHelpers
Use `similarHelpers` for **related but different helpers**:

**Example**: FilterMap combines Map and Filter functionality:
```yaml
similarHelpers:
  - core#slice#map        # Related transformation helper
  - core#slice#filter     # Related filtering helper
```

**Example**: Cross-package equivalents:
```yaml
similarHelpers:
  - parallel#slice#map    # Parallel version in different package

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samber/lo](https://github.com/samber/lo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
