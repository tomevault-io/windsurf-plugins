---
trigger: always_on
description: **Tech Stack**: PHP 8.4, Tempest Framework (v2), HtmlViewTree, UnoCSS, Pico CSS, PHPUnit, PHPStan (level 7)
---

# Tempest Pico — Copilot Instructions

## Quick Start

**Tech Stack**: PHP 8.4, Tempest Framework (v2), HtmlViewTree, UnoCSS, Pico CSS, PHPUnit, PHPStan (level 7)

**Yohn's Fork of PicoCSS is used** - see `https://yohn.github.io/PicoCSS/` and `https://github.com/Yohn/PicoCSS/`

**HtmlViewTree**: a alternative to string-based template engines, see **Html View Tree Builder** below.

**QA Workflow**:
```bash
composer qa           # fmt + phpunit + lint (use this before PRs)
composer phpunit      # Run tests with detailed output
composer fmt          # Format with mago
composer lint         # Lint with mago (auto-fixes)
pnpm unocss --watch   # Watch CSS during development
```

**Key Namespaces**:
- `TempestPico\` → `/src/` (main application)
- `Tests\` → `/tests/` (PHPUnit tests)

**PHPStan**: Level 7 strict checking on `src/`, `app/`, `tests/`

---

## Type Hints and common abbreviations

- `MD` → Markdown (GitHub flavored), helper: `MD()`
- `IMD` → Inline Markdown (no Block elements like `<p>`), helper: `IMD()`
- If you see `MD`/`IMD` as Type hint it means the corresponding helper is used on the given `string`   
- (Component-)`Content` one of Type:
  - `string|Stringable`  →  auto-escaped for safe output in HTML
  - `HtmlString`  →  raw HTML, must be used with care -> use `Html(…)` helper instead
  - `Component`  →  any class implementing `IsComponent` (renders to HTML via `getViewTree()`)
  - `View`  →  ! not implemented ! - any other Tempest View (should work - with Issues)
- `VT`/`HtmlViewTree`  →  A Tree of the above types, use `VT()` or `Html(…)` helper to create it


---

## Project Principles

### 1. **Semantic HTML/CSS First**
- Prefer semantic HTML elements over generic divs
- Use Pico CSS utilities + UnoCSS for modifiers (BEM workflow: B for semantic, M/E for utilities)
- Avoid Tailwind-style utility-heavy markup; maintain readability
- Prefer HtmL View Tree Builder over template strings

### 2. **IDE & Static Analysis Clarity**
- All template variables must be IDE-discoverable (no `$$key` magic, explicit type hints)
- Property promoted constructors for class dependency injection
- PHPStan level 7: catches undefined classes, type errors, null access issues
- Common issue: **undefined exception classes** → Always create Exception classes in `Exception/`

### 3. **Component Architecture**
- Components live in `src/Components/` with a `.php` (logic and a `getViewTree()`) and `.view.php` (that start rendering) pair
- Components should also provide:
  - a example returning `VT()` or `Html()` (`src/Components/Examples/*.php`)
  - a description / usage notes (`src/Components/Examples/*.md`)
- Extend `Component` base class and implement `IsComponent` interface

### 4. **Html View Tree Builder**
- **Purpose**: Build HTML trees programmatically without template strings
- **Key Methods**:
  - `__invoke(tag, content[], attributes[])` → Create HTML element (validates tag)
  - `customTag(tag, content[], attributes[])` → Custom tag (no validation)
  - `appendContent()` → Add children to current node
  - `render()` → Convert tree to `HtmlString`
- **Common Patterns**:
  ```php
  Html('div')
    ('h1', ['Headline']);
  ```
- **Known Pitfalls**:
  - **Void tags** (`<br>`, `<hr>`, etc.) cannot have content → throws `VoidWithContent` exception

---

## Common Tasks

### Add a New Component

0. use `src/Components/Accordion.php` and `src/Components/Table.php` as reference
1. Create `src/Components/*.php` (logic + dependencies)
2. Create `src/Components/*.view.php` (only needs: `<?= $this->toHtml();`)
3. Implement `IsComponent` and extend `Component`
4. Use it in other components or controllers
5. Add the `class::name` to `src/Components/Doc.php::COMPONENTS`

### Add Examples, short Documentation and Tests for a Component

- Place usage examples and a additional Notes (`*.md` file) in `src/Components/Examples`
- The Notes don't need more examples or be long. Just known Bugs or a "Read more @"-Link is fine.
- Use the example in test at `test/`.

### Add a Helper Function
- Place HtmlViewTree Helpers in `src/Support/Html/functions.php` (auto-loaded via composer.json)
- Example: `Html()` is aliased to `HtmlViewTree` constructor
- Always import with `use function TempestPico\Support\...`

### Run Tests Before PR
```bash
composer qa
```
This ensures:
- Code is formatted (mago fmt)
- All tests pass (PHPUnit)
- Linting passes (mago lint --fix)

### Debug HtmlViewTree Issues
- Use `render()` return value type (must be `HtmlViewTree`)
- Verify void tags never get `appendContent()` called
- PHPStan will catch missing exception definitions immediately

---

## Architecture Overview

```
src/
  Components/         # Reusable UI components
    *.php             # Logic
    *.view.php        # Render (returns HtmlString)
    Examples/         
      *.php           # Example (used in Tests)
      *.md            # Notes / Documentation
  Layout/             # Page layout (header, nav, footer) and configs
    Page/             # Base Html for pages
  Page/               # Route handlers / page builders / Controllers
  Support/
    Html/             # HTML View Tree Builder, Helper
      Exception/      # Custom exceptions (InvalidTag, VoidWithContent, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a-kla/tempest-pico](https://github.com/a-kla/tempest-pico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
