---
trigger: always_on
description: Fluent is a type-safe, composable HTML generation library for Go. Every HTML element is a Go package (e.g. `div`, `a`, `input`). Elements are constructed with `New()`, configured with chainable methods, and rendered with `Render()`.
---

# Fluent - HTML Generation for Go

Fluent is a type-safe, composable HTML generation library for Go. Every HTML element is a Go package (e.g. `div`, `a`, `input`). Elements are constructed with `New()`, configured with chainable methods, and rendered with `Render()`.

## Build & Test

```bash
go build ./...
go test ./...
go vet ./...
```

## Lint With Flint

After writing or editing Fluent code, run Flint. It catches Fluent-specific misuse that `go vet` cannot, and every diagnostic includes a `fix:` field with the corrected code - read the fix, apply it, and re-run until clean.

```bash
go install github.com/jpl-au/flint/cmd/flint@latest
flint ./...
```

What Flint flags:

- **Hallucinated APIs.** Every function, method, and type reference is validated against a generated registry. Catches calls like `node.Fragment()`, `div.New().Href(...)` (wrong element), `inputtype.Telephone` (does not exist).
- **Raw strings where typed constants are required.** `input.New().Type("email")` is flagged with a fix pointing at `inputtype.Email`. See the "Never use raw strings where typed constants exist" section below for the full rule.
- **Unsafe `Static()` and `RawText()`.** `Static()` requires a string literal so the JIT can pre-render it. `RawText()` does not HTML-escape. Both are flagged when called with a variable.
- **`New().Method()` redundancy.** Flags `div.New().Text("x")` and suggests `div.Text("x")` directly. See "Choosing the Right Constructor" below.
- **Typed constructor opportunities.** Suggests `ul.Items(...)` over `ul.New(li...)`, `tr.Cells(...)` over `tr.New(td...)`, and similar type-safe alternatives when children are uniform.
- **`SetAttribute()` misuse.** Flags chaining after `SetAttribute()` (it returns void), and flags usage where a typed method exists (`.Class()` instead of `.SetAttribute("class", ...)`).
- **Reserved keyword imports.** `select` → `dropdown`, `main` → `primary`, `var` → `variable`.

Use `-info` to look up an element's full API before writing against it - constructors, methods, typed parameters, and the constants each method accepts:

```bash
flint -info div          # everything about <div>
flint -info input        # every typed constant for every method
flint -info ol           # list constructors and typed variants
```

Exit codes: `0` clean, `1` diagnostics found, `2` usage or I/O error. Treat `1` as "fix and re-run", not "done".

## Methods That Do NOT Exist

**CRITICAL:** These methods do not exist anywhere in Fluent. Do not use them. Do not hallucinate them. They have never existed.

| Non-existent method | What to use instead |
|---------------------|---------------------|
| `.Attr()` | Use the dedicated typed method (e.g. `.Class()`, `.Href()`, `.Src()`) |
| `.SetAttr()` | Use `.SetAttribute()` for custom attributes only |
| `.Attribute()` | Use the dedicated typed method or `.SetAttribute()` |
| `.Attrs()` | No bulk attribute setter exists - set each attribute individually |
| `.WithAttr()` | Use the dedicated typed method or `.SetAttribute()` |

**The correct approach to setting attributes has three levels:**

1. **Dedicated typed methods (use first)** - Every standard HTML attribute has a chainable method. For example: `.Class()`, `.ID()`, `.Href()`, `.Src()`, `.Alt()`, `.Title()`, `.Disabled()`, `.Required()`, `.Placeholder()`, `.Name()`, `.Value()`, `.Type()`, etc.
2. **SetAria(key, value)** - For ARIA attributes. Automatically adds the `aria-` prefix.
3. **SetData(key, value)** - For data attributes. Automatically adds the `data-` prefix.
4. **SetAttribute(key, value)** - Only for truly custom or non-standard attributes (e.g. Alpine.js directives, HTMX attributes).

```go
// WRONG - these methods do not exist
div.New().Attr("class", "container")           // NO
div.New().SetAttr("id", "main")                // NO
button.New().Attribute("disabled", "")         // NO

// RIGHT - use dedicated typed methods
div.New().Class("container")                   // YES
div.New().ID("main")                           // YES
button.New().Disabled()                        // YES

// RIGHT - use SetAria for ARIA attributes
button.New().SetAria("label", "Close dialog")  // YES - renders aria-label="Close dialog"

// RIGHT - use SetData for data attributes
div.New().SetData("id", "123")                 // YES - renders data-id="123"

// RIGHT - use SetAttribute only for custom/non-standard attributes
div.New().SetAttribute("x-on:click", "handler")  // YES - Alpine.js directive
div.New().SetAttribute("hx-get", "/items")        // YES - HTMX attribute
```

**Important:** `SetAttribute()` does not return the element for chaining. `SetAria()` and `SetData()` do return the element for chaining.

## Common Mistakes - Read This First

**CRITICAL:** These are the most frequent errors. Every one of them causes a compile failure.

### 1. Never use raw strings where typed constants exist

Fluent uses `[]byte`-based types for enumerated HTML attribute values. The method signature **will not accept a plain string**. You must import the constant package and use its exported variable.

```go
// WRONG - does not compile

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpl-au/fluent](https://github.com/jpl-au/fluent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
