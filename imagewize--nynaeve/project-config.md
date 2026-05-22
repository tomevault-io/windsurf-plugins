---
trigger: always_on
description: This file provides GitHub Copilot with context about the Nynaeve WordPress theme development practices and conventions.
---

# GitHub Copilot Instructions - Nynaeve Theme

This file provides GitHub Copilot with context about the Nynaeve WordPress theme development practices and conventions.

## Project Overview

**Nynaeve** is a modern WordPress theme built on:
- **Sage 11** framework with Laravel Blade templating
- **Tailwind CSS 4** with custom design system
- **Acorn** (Laravel for WordPress) for advanced PHP features
- **Vite** for fast development and HMR
- **WooCommerce** integration with custom quote-based system
- **imagewize/sage-native-block** package for custom block development

## Block Development Philosophy (CRITICAL)

**PREFERRED APPROACH**: Build blocks using **InnerBlocks** with native WordPress blocks whenever possible.

### Key Principles
- **Maximum User Control**: Let users select styles, font sizes, and formatting via block toolbar/inspector
- **Avoid Hardcoded Classes**: Never hardcode styling classes in templates (e.g., `is-style-*`, `has-*-font-size`)
- **Native WordPress Blocks**: Use core blocks (Button, Heading, Paragraph, Image) within custom containers
- **Block Toolbar First**: Users should access all styling options via WordPress native controls
- **Minimal Inspector Controls**: Only add custom controls when absolutely necessary

### Block Development Options (In Order of Preference)

1. **Sage Native Blocks with InnerBlocks (MOST PREFERRED)**
   - Content-focused blocks with images, headings, text, buttons
   - Users need full typography control
   - Want clean sidebar with no custom inspector controls

2. **Sage Native Blocks with Custom Controls (Use Sparingly)**
   - Need dynamic frontend JavaScript interactivity
   - Complex data structures that don't map to core blocks

3. **ACF Composer Blocks (Special Cases Only)**
   - Need complex custom field types (repeaters, relationships)
   - Server-side rendering is critical
   - Editing must be rigid/controlled

## Development Commands

### Local Development
```bash
# Start development with HMR (use HTTP not HTTPS for HMR)
npm run dev

# Build for production
npm run build

# Code quality
composer pint
```

### Acorn Commands (Trellis VM Required)
```bash
# Enter Trellis VM
trellis vm shell

# Create Sage Native Block (InnerBlocks approach)
wp acorn sage-native-block:create

# Create ACF Composer Block (when InnerBlocks won't work)
wp acorn acf:block MyBlock

# Clear ACF cache
wp acorn acf:clear
```

**Note on Config Publishing (v2.0.1+):**
- As of `imagewize/sage-native-block` v2.0.1, config publishing is **no longer required**
- Package now works out-of-the-box without manual setup
- Projects with previously published configs will continue to work (package uses published config if present)

## Code Standards

### Block Standards (block.json)
```json
{
  "name": "imagewize/block-name",
  "category": "imagewize",
  "textdomain": "imagewize"
}
```

### InnerBlocks Template Patterns
Always use **real, publishable content** in templates (not placeholders):

```jsx
const TEMPLATE = [
  ['core/image', { className: 'card__image' }],
  ['core/heading', {
    level: 3,
    content: 'Professional WordPress Development'  // Real content!
  }],
  ['core/paragraph', {
    content: 'Transform your website with modern development practices.'
  }],
  ['core/buttons', { className: 'card__buttons', layout: { type: 'flex' } }, [
    ['core/button', { text: 'Get Started' }],
    ['core/button', { text: 'Learn More' }]
  ]]
];
```

### Button Styling (CRITICAL)
WordPress doesn't reliably apply className to individual buttons. Use container approach:

```jsx
// ✅ CORRECT - className on buttons container
['core/buttons', {
  className: 'my-buttons-container',
  layout: { type: 'flex' }
}, [
  ['core/button', { text: 'Click Me' }]
]]
```

```css
/* Target buttons via container */
.my-block .my-buttons-container .wp-block-button .wp-block-button__link {
  background-color: black;
}
```

## File Structure

```
nynaeve/
├── app/                    # PHP application code
│   ├── Blocks/            # ACF Composer blocks
│   └── Providers/         # Service providers
├── resources/
│   ├── css/               # Tailwind CSS styles
│   ├── js/
│   │   └── blocks/        # Sage Native blocks
│   └── views/             # Blade templates
├── config/                # Configuration files
└── public/build/          # Built assets
```

## Theme Configuration

### Colors (Tailwind)
Theme uses custom color palette defined in `tailwind.config.js` and exposed via `theme.json`:
- `primary` - Primary brand color (blue)
- `main` - Main text/dark color (dark gray/black)
- `base` - Base/background white
- `secondary` - Secondary gray
- `tertiary` - Tertiary background (light gray)

**Reference:** See `resources/css/app.css` for CSS custom properties or `config/theme.json` for WordPress block editor colors.

### Typography
- **Headings**: Montserrat font family
- **Body**: Open Sans font family

## Layout Conventions (WordPress-Native Approach)

We use the **Twenty Twenty-Five** layout system - WordPress's modern native layout with **minimal custom CSS**.

### How It Works
1. `theme.json` sets `"useRootPaddingAwareAlignments": true`
2. `theme.json` sets root padding via `styles.spacing.padding`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imagewize/nynaeve](https://github.com/imagewize/nynaeve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
