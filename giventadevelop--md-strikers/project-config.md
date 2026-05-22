---
trigger: always_on
description: This guide provides standards for creating consistent and maintainable UI components across the application UI
---

# UI Component Style Guide

This guide provides standards for creating consistent and maintainable UI components across the application UI
.

---

## 1. Page & Content Layout

### Page Container

- **Rule:** Use `max-w-5xl mx-auto px-8 py-8` for main page containers.
- **Purpose:** Enforces a consistent 80% width and center alignment on desktop views.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent page layout
  <div className="max-w-5xl mx-auto px-8 py-8">
    {/* Page content goes here */}
  </div>
  ```

### Content Card

- **Rule:** Use `bg-white rounded-lg shadow-md p-6` for containers that wrap main content sections (tables, forms, etc.).
- **Purpose:** Creates a consistent, elevated card-based layout for content.
- **Example:**
  ```tsx
  // ✅ DO: Use a styled container for content sections
  <div className="bg-white rounded-lg shadow-md p-6">
    {/* Table, list, or form content */}
  </div>
  ```

### Centered Card Grid Layout

- **Rule:** Use CSS modules with flexbox for centered card grids (Featured Guests, Contact Information, Program Directors, Gallery thumbnails, Team members).
- **Purpose:** Ensures cards are perfectly centered regardless of the number of items, preventing left-aligned layouts when items don't fill the full width.
- **Pattern:** Create a CSS module file (e.g., `CenteredCardGrid.module.css`) with:
  - Flexbox container with `justify-content: center`
  - Responsive `max-width` calculations based on number of columns
  - Fixed or calculated widths for card items
- **Example:**
  ```css
  /* Centered Card Grid */
  .centeredCardGrid {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    width: 100%;
    justify-content: center;
    align-items: flex-start;
    margin: 0 auto;
  }

  /* Desktop: 3 columns */
  @media (min-width: 1024px) {
    .centeredCardGrid {
      max-width: calc(3 * 350px + 2 * 1rem);
    }
    .cardItem {
      width: 350px;
      max-width: 350px;
    }
  }

  /* Tablet: 2 columns */
  @media (min-width: 768px) and (max-width: 1023px) {
    .centeredCardGrid {
      max-width: calc(2 * 350px + 1 * 1rem);
    }
    .cardItem {
      width: calc((100% - 1rem) / 2);
      max-width: calc((100% - 1rem) / 2);
    }
  }

  /* Mobile: 1 column */
  @media (max-width: 767px) {
    .centeredCardGrid {
      max-width: 100%;
    }
    .cardItem {
      width: 100%;
      max-width: 100%;
    }
  }
  ```
- **Usage:**
  ```tsx
  // ✅ DO: Use CSS module for centered card grids
  import cardGridStyles from './CenteredCardGrid.module.css';

  <div className={cardGridStyles.centeredCardGrid}>
    {items.map((item) => (
      <div key={item.id} className={cardGridStyles.cardItem}>
        {/* Card content */}
      </div>
    ))}
  </div>

  // ❌ DON'T: Use standard grid without centering
  <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {/* Cards will be left-aligned when not filling full width */}
  </div>
  ```
- **References:**
  - See `src/app/events/[id]/CenteredCardGrid.module.css` for implementation
  - See `src/app/events/[id]/GalleryThumbnails.module.css` for gallery pattern
  - See `src/components/TeamSection.module.css` for team member pattern

---

## 2. Forms

### Input Fields

- **Rule:** Use the following classes for consistent input field styling.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent input field styling
  <input
    type="text"
    className="mt-1 block w-full border border-gray-400 rounded-xl focus:border-blue-500 focus:ring-blue-500 px-4 py-3 text-base"
  />
  ```

### Labels

- **Rule:** Use the following classes for consistent label styling.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent label styling
  <label className="block text-sm font-medium text-gray-700">
    Field Label
  </label>
  ```

### Checkboxes

- **Rule:** Use the `custom-checkbox` implementation for a larger, more visible checkbox with a custom tick mark.
- **Click Handling:** Always include `onClick={(e) => e.stopPropagation()}` on the `input` to prevent unintended event bubbling, especially inside clickable table rows or containers.
- **Example:**
  ```tsx
  // ✅ DO: Use consistent checkbox styling with stopPropagation
  <label className="flex flex-col items-center">
    <span className="relative flex items-center justify-center">
      <input
        type="checkbox"
        className="custom-checkbox"
        checked={isChecked}
        onChange={handleChange}
        onClick={(e) => e.stopPropagation()}
      />
      <span className="custom-checkbox-tick">
        {isChecked && (
          <svg className="w-6 h-6 text-black" fill="none" stroke="currentColor" strokeWidth="4" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" d="M5 13l5 5L19 7" />
          </svg>
        )}
      </span>
    </span>
    <span className="mt-2 text-xs text-center select-none break-words max-w-[6rem]">Checkbox Label</span>
  </label>
  ```

- **Checkbox Group Layout:**
  ```tsx
  // ✅ DO: Use a CSS grid for checkbox group layout
  <div className="custom-grid-table mt-4" style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '0.5rem' }}>
    {/* Checkbox items */}
  </div>
  ```

- **Required CSS (`globals.css`):**
  ```css
  .custom-checkbox {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
