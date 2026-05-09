---
trigger: always_on
description: This project is intentionally simple. The goal is a **production-ready, maintainable website with zero build tools, zero dependencies to install, and zero framework complexity**. Anyone with basic HTML knowledge should be able to edit it. It should work indefinitely without updates, run on any host including shared hosting, and be easy to hand off.
---

# Pure HTML + Bootstrap Website

## Project Philosophy

This project is intentionally simple. The goal is a **production-ready, maintainable website with zero build tools, zero dependencies to install, and zero framework complexity**. Anyone with basic HTML knowledge should be able to edit it. It should work indefinitely without updates, run on any host including shared hosting, and be easy to hand off.

**When in doubt, do less. Simpler is always preferred.**

---

## Stack

| Concern | Tool | Why |
|---|---|---|
| Structure | Plain HTML5 | No build step, universally understood |
| Styling | Bootstrap 5 via CDN | Stable, small, well-documented |
| Icons | Bootstrap Icons via CDN | Consistent with Bootstrap, lightweight |
| JavaScript | Vanilla JS only | No npm, no bundler, no framework |
| Fonts | System font stack | Zero external requests, fast |

### CDN URLs (pin these versions, do not auto-upgrade)

```html
<!-- Bootstrap CSS -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">

<!-- Bootstrap Icons -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">

<!-- Bootstrap JS (only include if interactive components are needed) -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
```

> **Important:** Always pin exact versions. Never use `@latest`. CDN URLs should only change when there is a deliberate decision to upgrade.

---

## Project Structure

```
/
├── index.html          # Homepage
├── about.html          # About page (if needed)
├── contact.html        # Contact page (if needed)
├── css/
│   └── style.css       # Custom overrides only — keep this minimal
├── js/
│   └── main.js         # Custom JS only — keep this minimal
├── img/
│   └── ...             # All images, optimized before committing
└── CLAUDE.md           # This file
```

No subdirectories beyond the above unless clearly justified.

---

## HTML Rules

- Every page must have a valid `<!DOCTYPE html>` declaration
- Every page must have `<html lang="en">` (adjust language code as needed)
- Every page must have a unique, descriptive `<title>` tag
- Every page must have a `<meta name="description">` tag
- Every page must include `<meta name="viewport" content="width=device-width, initial-scale=1">`
- Semantic HTML elements are preferred: `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`
- Never use `<div>` where a semantic element is appropriate
- All images must have descriptive `alt` attributes — never leave `alt` empty unless purely decorative
- All pages must share the same `<header>` and `<footer>` structure for consistency

### Page Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="Page description here">
  <title>Page Title — Site Name</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
  <link rel="stylesheet" href="css/style.css">
</head>
<body>

  <header>
    <nav class="navbar navbar-expand-lg navbar-light bg-white border-bottom">
      <!-- nav content -->
    </nav>
  </header>

  <main>
    <!-- page content -->
  </main>

  <footer class="bg-light border-top py-4 mt-5">
    <!-- footer content -->
  </footer>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
  <script src="js/main.js"></script>
</body>
</html>
```

---

## CSS Rules

- Bootstrap utility classes are preferred over writing custom CSS
- `style.css` is for **overrides and custom components only** — not for reimplementing what Bootstrap already provides
- Never use `!important` unless absolutely unavoidable — if you need it, reconsider the approach
- CSS custom properties (variables) are preferred for brand colors and repeated values
- Mobile-first. Use Bootstrap's responsive utilities. Test at 375px, 768px, and 1280px minimum
- No inline styles except for truly dynamic values set via JavaScript

### Recommended CSS Variables

```css
:root {
  --color-primary: #your-brand-color;
  --color-secondary: #your-secondary-color;
  --color-text: #212529;
  --color-muted: #6c757d;
  --font-body: system-ui, -apple-system, sans-serif;
}
```

---

## JavaScript Rules

- Vanilla JS only. No jQuery, no Alpine, no Vue, no React
- Bootstrap's built-in JS handles modals, dropdowns, accordions — use it before writing custom JS
- `main.js` should be small. If it grows beyond ~100 lines, question whether the feature belongs in a pure HTML site
- No JS should be required for the page to be readable and navigable — JS is enhancement only
- Always use `addEventListener`, never inline `onclick` attributes in HTML

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [commongoodguy/ai-just-html](https://github.com/commongoodguy/ai-just-html) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
