---
trigger: always_on
description: **Always update code comments, documentation, READMEs, and the demo page (`index.html`) whenever a feature is modified or added.** This includes:
---

# Zephyr Framework — Agent & Engineering Standards

## Documentation Rule

**Always update code comments, documentation, READMEs, and the demo page (`index.html`) whenever a feature is modified or added.** This includes:

- JSDoc comments on public methods and classes
- `README.md` usage examples and feature list
- `AGENTS.md` stack layout if architecture changes
- `index.html` demo if a component's API changes
- `claude-roadmap.md` — mark items complete or add new items as they emerge

## Engineering Standards

All code in this project must follow these principles:

### DRY (Don't Repeat Yourself)
- Extract shared logic into base class methods or utility functions
- Shared CSS patterns use common selectors or CSS custom properties
- No copy-pasted blocks — if a pattern appears twice, abstract it

### Modular
- Each component is a self-contained custom element class
- Components communicate via DOM events, not direct references
- CSS is scoped to component selectors (`z-*`)
- Framework files are separated from demo files

### Maintainable
- Clear naming conventions (see below)
- JSDoc on all public methods
- Minimal coupling between components
- Progressive enhancement — components degrade gracefully

## Naming Conventions

| Category | Convention | Example |
|----------|-----------|---------|
| Custom elements | `z-` prefix, kebab-case | `z-accordion`, `z-modal` |
| CSS classes | kebab-case | `carousel-controls` |
| Data attributes | `data-` prefix, kebab-case | `data-open`, `data-active` |
| JS classes | PascalCase with `Z` prefix | `ZAccordion`, `ZModal` |
| Private members | underscore prefix | `_currentIndex`, `_transition()` |
| CSS variables | `--z-` prefix | `--z-transition-duration` |
| Events | lowercase, no prefix | `change`, `open`, `close` |

## Testing Standards

- Every component must have unit tests covering: registration, attribute handling, state transitions, event dispatch
- Form-associated components need integration tests verifying `FormData` participation
- Accessibility: validate ARIA attributes and keyboard navigation
- Test harness should be HTML-based (no Node dependency) to match the zero-JS philosophy
- Tests live in a `tests/` directory
- `npm test` runs every `tests/test-*.html` page headlessly via `tests/run-ci.js` (Playwright Chromium, devDependency only); the same suite runs in GitHub Actions (`.github/workflows/ci.yml`). Pages remain directly openable in a browser.
- Inside test scripts, never write a literal `</script>` in a string — escape it as `<\/script>` or the HTML parser truncates the script element

## Code Hygiene

- No `innerHTML` for untrusted content — use `cloneNode()` or DOM manipulation
- All event listeners added in `connectedCallback` must be removed in `disconnectedCallback`
- Intervals and timeouts must be stored and cleared on disconnect
- Use CSS custom properties for all themeable values
- `formAssociated` only on components that participate in forms
- All components dispatch events for state changes

## Performance Standards

- All animations use GPU-accelerated properties (`transform`, `opacity`)
- No layout-triggering animations (`width`, `height`, `top`, `left`)
- Document-level event listeners are shared/delegated, not per-instance
- CSS transitions preferred over JS animations
- View Transitions API used with feature detection fallback

## Security Standards

- No `innerHTML` with user-supplied content
- Inline `onclick` handlers should be replaced with delegated event listeners for CSP compatibility
- Document security model for content injection in components
- Sanitize any content that flows from user input into the DOM

## State Attribute Conventions

| Attribute | Purpose | Used By |
|-----------|---------|---------|
| `data-open` | Binary open/closed state | Accordion, Select, Dropdown |
| `data-active` | Selected/current item in a set | Tabs, Carousel |
| `data-visible` | Visibility toggle | Toast |
| `data-value` | Item value for selection | Select options |

## Full Stack Layout

```
zephyr-framework/
├── zephyr-framework.js      # Core framework — Web Components (custom elements)
│   ├── ZephyrElement          # Base class (shared utilities, lifecycle, click-outside)
│   ├── ZAccordionItem         # Registered custom element for accordion items
│   ├── ZAccordion             # Collapsible sections via CSS Grid + ARIA
│   ├── ZModal                 # Native <dialog> wrapper with View Transitions + ARIA
│   ├── ZTabs                  # Tab panels with View Transitions + keyboard nav
│   ├── ZSelect                # Form-associated custom select (ElementInternals) + ARIA
│   ├── ZCarousel              # Slide viewer with autoplay + keyboard nav
│   ├── ZToast                 # Notification system (role=alert)
│   ├── ZDropdown              # Menu dropdown with click-outside + ARIA
│   ├── ZCombobox              # Filterable combobox with keyboard nav + ARIA
│   ├── ZDatepicker            # Enhanced native date input + formatted display
│   ├── ZInfiniteScroll        # IntersectionObserver-based infinite loading
│   ├── ZSortable              # Native Drag & Drop reorderable list

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daltlc/zephyr-framework](https://github.com/daltlc/zephyr-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
