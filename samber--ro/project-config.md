---
trigger: always_on
description: When adding a new method to the `ro` library, follow these guidelines to create or update documentation files in the `docs/data/` directory.
---

# Adding New Method Documentation

When adding a new method to the `ro` library, follow these guidelines to create or update documentation files in the `docs/data/` directory.

## File Structure

Documentation files follow the naming pattern `core-{method-name}.md` for core methods and `plugin-{plugin-name}-{method-name}.md` for plugins.

### Frontmatter Format

Each documentation file must start with a frontmatter section:

```yaml
---
name: methodName
slug: methodname
sourceRef: file.go#L123
type: core
category: filtering
signatures:
  - "func MethodName(params)"
  - "func (receiver *Type) MethodName(params)"
  - "func MethodNameI(params)"
  - "func MethodNameWithContext(params)"
playUrl: https://go.dev/play/p/EXAMPLE
variantHelpers:
  - core#slice#methodname
  - core#slice#methodnamei
  - core#slice#methodnamewithcontext
similarHelpers:
  - core#slice#filtermethodname
  - plugin#encoding-json#othermethodname
position: 0
---
```

### Frontmatter Fields

- **name**: The display name of the helper (PascalCase)
- **slug**: URL-friendly name (kebab-case, matches filename without `core-` prefix)
- **sourceRef**: Source file reference with line number (format: `operator_conditional.go#L123`). Line numbers drift whenever the referenced Go file is edited (added imports, new operators, reordered code...). After changing a `.go` file, re-check every `sourceRef` pointing into it: run `gopls symbols <file>` to list the file's operators/functions with their current line numbers, then update the affected `sourceRef` fields accordingly.
- **type**: `core`, `plugin`. The category must match the file name.
- **category**: The functional category. For plugins, the category must match the file name. Some plugins might have a sub-sub-category: in that case use "-" (eg: plugin > `encoding-json` or plugin > `samber-hot` or plugin > `logger-logrus`).
- **signatures**: Array of function signatures as strings. Do not list signatures from other type/category.
- **playUrl**: Go Playground URL with working example
- **variantHelpers**: Array of variant method names. Must contain at least the default method of named above. Variation XxxxWithContext or XxxxF or Xxxx2/3/4/5/... or XxxxI must be listed here. Don't list methods from other packages (type/category) of this library (must be similarHelpers instead).
- **similarHelpers**: Array of related helper names (leave empty if none). Eg: equivalent in other package/category, or helper composition (map vs filtermap), or method with callback (Last vs LastBy).
- **position**: Position in the list (0, 10, 20, 30...). Order must follow the order in source code. Helpers are grouped by type+category and displayed on a page. Position number is reset for each page.

## Content Structure

After the frontmatter, include:

1. **Brief description**: One sentence explaining what the helper does
2. **Code example**: Working Go code demonstrating usage
3. **Expected output**: Comment showing the result

```markdown
Brief description of what this helper does.

```go
obs := ro.Pipe[int, int](
    ro.Just(1, 2, 3, 4),
    ro.MethodName(example),
)

sub := obs.Subscribe(ro.PrintObserver[bool]())
defer sub.Unsubscribe()

// expected result
```
```

## Code Style Guidelines

- Use `Pipe` instead of `PipeX` variants.
- Use `NewObserver` or `NewObserverWithContext` instead of `Observer{...}`. Prefer NewObserver when no context is used.
- Example: `NewObserver(next, error, complete)` instead of `Observer{Next: next, Error: error, Complete: complete}`
- Use OnNext/OnNextWithContext/OnError/OnErrorWithContext/OnComplete/OnCompleteWithContext instead of NewObserver/NewObserverWithContext, when only a single callback is needed.

Multiple examples can be used for demonstration the method, such as edge cases. If multiple signatures/variants are grouped under this documentation, it could be useful to describe some (all?) of them. You can create examples by yourself or read _example_test.go files.

## Grouping Related Methods

**IMPORTANT: Distinguish between variants and similar helpers**

### Variants vs Similar Helpers

**Variants** should be grouped in a single file when:
- They are true variations of the same base functionality with **identical core behavior**
- They only differ by adding context (`WithContext`), index (`I`), or both (`IWithContext`) parameters
- They have the same fundamental purpose and behavior pattern
- Examples: `Filter()`, `FilterI()`, `FilterWithContext()`, `FilterIWithContext()`
- Examples: `CombineLatest()`, `CombineLatest2()`, `CombineLatest3()`, `CombineLatest4()`

**Similar helpers** should be documented as separate files when:
- They have **different core behavior** or functionality
- They solve different problems or use different algorithms
- They are composed differently (e.g., `Map` vs `FilterMap` vs `MapErr`)
- Examples: `BufferWhen` vs `BufferWithCount` (different buffering strategies), `CombineLatest` vs `CombineLatestWith` (different calling patterns)

### Guidelines for Method Grouping

**Group together as variants (single file):**
- Base method + context/index variants: `Method()`, `MethodI()`, `MethodWithContext()`, `MethodIWithContext()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samber/ro](https://github.com/samber/ro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
