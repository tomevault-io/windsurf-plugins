---
trigger: always_on
description: These instructions apply to all Astro theme work. Prioritize clean, reusable, accessible, fast, SEO-friendly code. Treat the theme as something that may be reused across multiple websites, not as a one-off implementation.
---

# Instructions for Astro Theme Development

These instructions apply to all Astro theme work. Prioritize clean, reusable, accessible, fast, SEO-friendly code. Treat the theme as something that may be reused across multiple websites, not as a one-off implementation.

## General Principles

* Prefer simple, maintainable Astro components over unnecessary abstractions.
* Keep the default Astro advantage: mostly static HTML, minimal JavaScript, and hydration only where needed.
* Do not add client-side JavaScript unless there is a clear user-facing reason.
* Avoid unnecessary dependencies. Before adding a package, check whether the same result can be achieved with Astro, HTML, CSS, or a small utility.
* Keep components reusable, documented, and easy to override.
* Use TypeScript where helpful, especially for props, content schemas, config objects, and reusable utilities.
* Favor progressive enhancement. The site should remain usable even if JavaScript fails.
* Keep markup clean, semantic, and easy to crawl.
* Never solve layout or behavior problems in a way that harms accessibility, SEO, or performance.
* Follow README layout as here https://github.com/andreialba/maria must include the title, preview image with a link to the preview URL, those cards with versions, preview link, short description of the theme, list of features, and how to set things up.
* Add MIT license under Andrei Alba

## Astro-Specific Guidelines

* Use `.astro` components for static and content-focused UI.
* Use islands/client hydration only when interactivity is required.
* Avoid `client:load` unless the component truly needs to run immediately.
* Prefer `client:visible`, `client:idle`, or no hydration when possible.
* Keep layout components responsible for page structure, shared metadata, global slots, and theme-level wrappers.
* Keep UI components small and focused.
* Use `Astro.props` with typed props where possible.
* Use content collections for structured content like posts, pages, projects, docs, testimonials, FAQs, and changelogs.
* Validate frontmatter with schemas instead of relying on loose optional fields.
* Keep route structure clean and predictable.
* Do not hardcode production URLs inside components. Use site config, constants, or environment-aware helpers.
* Make sure the theme works with a configurable `site` value in `astro.config.*`.

## Accessibility Requirements

* Use semantic HTML first. Do not use ARIA when a native HTML element solves the problem.
* Use landmarks properly: `header`, `nav`, `main`, `section`, `article`, `aside`, and `footer` where appropriate.
* Each page should have one clear `h1`.
* Preserve logical heading order. Do not skip heading levels for visual styling.
* All interactive elements must be keyboard accessible.
* Use real buttons for actions and real links for navigation.
* Every form control must have an associated label.
* Inputs, errors, help text, and validation states must be understandable to screen readers.
* Add visible focus styles. Never remove outlines without replacing them with an accessible focus state.
* Provide a skip link for keyboard users when the layout has repeated navigation.
* Use descriptive link text. Avoid vague text like “click here” or “read more” without context.
* Images must have useful `alt` text when meaningful.
* Decorative images should use empty alt text.
* Icons used as buttons or links must have accessible names.
* Ensure sufficient color contrast for text, icons, borders, and states.
* Do not rely on color alone to communicate meaning.
* Respect `prefers-reduced-motion`.
* Avoid auto-playing motion, carousels, or animations unless they are user-controlled and accessible.
* Modals, menus, accordions, tabs, dropdowns, and mobile navigation must handle focus, keyboard interaction, and escape/close behavior correctly.
* Test important templates with keyboard navigation and screen reader-friendly markup in mind.

## SEO Requirements

* Every page should have a unique, descriptive `<title>`.
* Every indexable page should have a useful meta description.
* Use a reusable SEO or Head component for metadata.
* Include canonical URLs where appropriate.
* Support Open Graph metadata for social sharing.
* Support Twitter/X card metadata where appropriate.
* Use absolute URLs for canonical and social image URLs.
* Configure `site` in `astro.config.*` so canonical URLs and sitemap generation work correctly.
* Include sitemap support for production themes.
* Include sensible robots handling.
* Avoid duplicate metadata across pages.
* Avoid duplicate content caused by inconsistent trailing slashes, canonical paths, or pagination.
* Use clean, descriptive URLs.
* Add structured data where useful, such as `WebSite`, `Organization`, `Article`, `BreadcrumbList`, `Product`, `FAQPage`, or `LocalBusiness`, depending on the theme.
* Do not add fake schema data. Structured data must match visible page content.
* Use proper heading structure to reflect the content hierarchy.
* Ensure important content is present in the HTML, not hidden behind client-only rendering.
* Use descriptive image filenames where possible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andreialba/quietpages](https://github.com/andreialba/quietpages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
