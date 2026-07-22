---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a **Laravel 12 + Preline UI + Tailwind CSS v4** project designed for building SaaS applications with a focus on beautiful UI components and dark mode support.

Main Laravel ecosystem packages & versions:
- php - 8.3.16
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- livewire/livewire (LIVEWIRE) - v3
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- tailwindcss (TAILWINDCSS) - v4
- tailwindcss/forms - v0.5.10
- **preline** (PRELINE UI) - v3.2.3

## Conventions
- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts
- Do not create verification scripts or tinker when tests cover that functionality and prove it works. Unit and feature tests are more important.

## Application Structure & Architecture
- Stick to existing directory structure - don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling
- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `npm run build`, `npm run dev`, or `composer run dev`. Ask them.

## Replies
- Be concise in your explanations - focus on what's important rather than explaining obvious details.

## Documentation Files
- You must only create documentation files if explicitly requested by the user.


=== preline-ui rules ===

## Preline UI Integration

This project uses **Preline UI v3** for pre-built UI components with Tailwind CSS v4. Preline is imported in `resources/js/app.js` and integrated via Vite.

### Component Initialization
- Preline components initialize automatically via `import 'preline'` in `app.js`
- The import includes dropdown menus, modals, tabs, accordions, and other interactive components
- No manual initialization required in most cases

### Component Layouts
- **Public pages**: Use `<x-layouts.public>` layout (e.g., `welcome.blade.php`, marketing pages)
- **Admin/dashboard**: Use `<x-admin.default>` or `<x-admin.split>` layouts for admin interfaces
- **Livewire**: Components use standard Livewire structure with Preline-styled elements

### Blade Components
- Reusable components in `resources/views/components/`:
  - `public/navbar.blade.php` - Fixed navigation with theme toggle
  - `public/footer.blade.php` - Footer for public pages
  - `admin/header.blade.php` - Admin dashboard header
- Check existing components in `resources/views/components/` before creating new ones

### Styling Guidelines
- All Preline components work seamlessly with Tailwind v4 utilities
- Preline variants are imported via `@import "../../node_modules/preline/variants.css"` in `app.css`
- Use custom dark mode classes (see Dark Mode section) instead of raw Tailwind for consistency


=== dark-mode rules ===

## Custom Dark Mode Implementation

This project uses a **custom class-based dark mode** system without `@preline/theme-switch`. Read `docs/DARK_MODE_GUIDE.md` for complete details.

### Architecture
- **JavaScript handler**: `resources/js/app.js` contains `window.HSThemeAppearance` API
- **Custom classes**: `resources/css/app.css` defines reusable dark mode classes
- **FOUC prevention**: Inline script in `resources/views/partials/head.blade.php` sets theme before render
- **Persistence**: Theme preference stored in `localStorage` as `hs_theme` (light/dark)

### Theme Toggle Implementation
- Toggle buttons use `data-hs-theme-click-value="dark"` or `data-hs-theme-click-value="light"`
- Example in `resources/views/components/public/navbar.blade.php` (lines 23-50)
- Buttons automatically show/hide based on active theme
- Can place theme toggle anywhere - it will work globally

### Custom Dark Mode Classes (CRITICAL)
Always use these custom classes for consistent dark mode support:

**Backgrounds:**
- `.bg-primary` - Main background (white/neutral-900)
- `.bg-secondary` - Secondary background (gray-50/neutral-800)
- `.section-bg` - Section backgrounds (gray-50/neutral-900)

**Text:**
- `.text-primary` - Main text (gray-900/white)
- `.text-secondary` - Secondary text (gray-600/gray-300)
- `.text-muted` - Muted text (gray-500/gray-400)

**Borders:**
- `.border-primary` - Standard borders (gray-200/neutral-700)

**Components:**
- `.feature-card` - Marketing feature cards with hover effects and dark mode
- `.icon-container-{color}` - Icon containers (blue, green, purple, orange) with dark mode
- `.icon-{color}` - Colored icons that adapt to dark mode

**Example Usage:**
```blade
<div class="bg-primary">
    <h1 class="text-primary">Title</h1>
    <p class="text-secondary">Description text</p>
</div>
```

### When Creating New Components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awankumay/Laravel-PrelineUI](https://github.com/awankumay/Laravel-PrelineUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
