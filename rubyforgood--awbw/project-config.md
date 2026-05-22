---
trigger: always_on
description: <!-- Keep code style rules in sync with CLAUDE.md -->
---

# This is a Ruby on Rails application.
<!-- Keep code style rules in sync with CLAUDE.md -->

For project overview, tech stack, architecture reference (models, controllers, services, testing), and more, read `AGENTS.md`.

## Setup

Full setup (bundle, npm, database create/migrate/seed):
```
bin/setup
```

If you just need frontend dependencies:
```
npm ci
```

## AI Instruction Files

When the user says "AI files", "AI instructions", "tell AI to", or "remember to always", these are the files.
If you notice the user repeatedly correcting the same pattern, suggest adding it to the AI files with a concrete proposal.

| File | Purpose |
|---|---|
| `CLAUDE.md` | Coding rules and conventions (this file) |
| `AGENTS.md` | Architecture reference + project details |
| `.github/copilot-instructions.md` | Coding rules for Copilot (duplicated from CLAUDE.md — keep in sync) |
| `ai/` | Shell script shortcuts for common dev tasks |

## Related Files

When changing a model or controller, check whether these related files need updates:

| If you change... | Also check... |
|---|---|
| Model | Decorator, policy, factory, model spec |
| Controller | Policy, request spec, routing spec, views |
| View | System spec, Stimulus controller (if interactive) |
| Service | Service spec |
| Decorator | Decorator spec |
| Mailer (add/remove) | Mailer spec, mailer preview (follow existing patterns) |
| Add/remove model, concern, service, or gem | AGENTS.md |

## Code Style

- Use modern Ruby syntax
- Prefer early returns and guard clauses
- Avoid unnecessary and/or complex conditionals
- Prefer constants and scopes over magic strings
- Use safe navigation (`&.`) where appropriate
- Use `presence` over blank checks
- Use `Arel.sql` for raw SQL in order clauses
- Avoid `update_all` unless explicitly intended
- Prefer service objects under app/services/
- Prefer POROs over concerns when possible
- Use `after_commit` instead of `after_save` for side effects

## RuboCop (rubocop-rails-omakase)

This project uses rubocop-rails-omakase. All code MUST follow these rules:

### Strings
- **Always use double quotes** for strings: `"foo"` not `'foo'`

### Spacing
- **Spaces inside array brackets:** `[ a, b, c ]` not `[a, b, c]` (empty arrays: `[]`)
- **Spaces inside hash braces:** `{ a: 1, b: 2 }` not `{a: 1}` (empty hashes: `{}`)
- **Spaces inside block braces:** `foo { bar }` not `foo {bar}` (empty blocks: `foo { }`)
- **No spaces inside parens:** `foo(bar)` not `foo( bar )`
- **No spaces inside reference brackets:** `hash[:key]` not `hash[ :key ]`
- **Space before block braces:** `foo { }` not `foo{ }`

### Commas
- **No trailing commas** in arrays, hashes, or method arguments

### Indentation
- **2-space indentation**, no tabs
- **Consistent indentation** at normal level — do NOT indent methods under `private`/`protected`
- **Align `end` with the variable** in assignments:
  ```ruby
  result = if condition
    value
  end
  ```
- **Align `when` with `end`**, not with `case`

### Whitespace
- **No trailing whitespace** on any line
- **No trailing blank lines** at end of file
- **No empty lines** inside class, module, method, or block bodies

### Syntax
- **Use `%w[]` and `%i[]`** with square bracket delimiters (not parens)
- **Use modern hash syntax:** `{ key: value }` not `{ :key => value }`
- **No redundant returns** — omit `return` on last expression
- **Use `flat_map`** instead of `.map { }.flatten`
- **No redundant `.to_s`** inside string interpolation
- **Use `Foo.method`** not `Foo::method` for method calls
- **No parentheses around conditions:** `if foo` not `if (foo)`
- **No semicolons** to separate statements

## Casing

- **Use sentence case** for UI labels, headings, and display text — not title case
- "Age range" not "Age Range", "Art type" not "Art Type"
- Use `.underscore.humanize` to convert PascalCase model/type names to sentence case (e.g., `"AgeRange".underscore.humanize` → `"Age range"`)
- Avoid `.titleize` for user-facing labels — it produces title case
- **Exception:** when a category type name prefixes a category name (e.g., "Age Range: 3-5"), use `.titleize` for the prefix

## HTML/ERB Formatting

### Tag Attributes
- **Closing `>` on same line as last attribute** — do not put `>` on its own line
- When attributes span multiple lines, keep the closing `>` with the last attribute
- Example (GOOD):
  ```erb
  <div class="relative z-10 w-full bg-white text-gray-800 py-2 px-4"
       id="dropdown">
  ```
- Example (BAD):
  ```erb
  <div class="relative z-10 w-full bg-white text-gray-800 py-2 px-4"
       id="dropdown"
  >
  ```

## JavaScript

- ES6+ syntax, ESM imports/exports, `const`/`let` (no `var`)
- Use `const` for fixed values — not `SCREAMING_SNAKE_CASE` constants (e.g., `const styleId = "foo"` not `const STYLE_ID = "foo"`)
- **Strongly prefer Stimulus** for JavaScript behavior — do not write raw/inline JS or jQuery
- **Always use Tailwind CSS** utility classes for styling — do not write custom CSS unless absolutely necessary
- **Prefer Font Awesome (free)** icons over inline SVGs — use `icon("fa-solid fa-foo")` helper. Inline SVGs are acceptable when a specific icon design is preferred.
- Prefer Turbo for navigation and form submissions before reaching for Stimulus
- Controller naming: `[name]_controller.js`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rubyforgood/awbw](https://github.com/rubyforgood/awbw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
