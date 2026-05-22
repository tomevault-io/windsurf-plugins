---
trigger: always_on
description: Icon styling standards and patterns for consistent iconography across the application
---


# Icon Standards and Styling Guide

This guide defines the standard patterns for displaying icons across the application, ensuring visual consistency and proper sizing.

---

## **Icon Library**

- **Primary Library:** Inline SVG icons using Heroicons pattern (Tailwind CSS compatible)
- **ViewBox:** Always use `viewBox="0 0 24 24"` for consistent scaling
- **Stroke Style:** Use `fill="none" stroke="currentColor"` with `strokeWidth={2}` for outline icons
- **Fill Style:** Use `fill="currentColor"` for solid icons when needed

---

## **Standard Icon Sizes**

### **Large Feature Icons (Event Cards, Feature Sections)**
- **Container:** `w-14 h-14` (56px × 56px)
- **Icon:** `w-10 h-10` (40px × 40px)
- **Container Style:** `rounded-xl` (12px border radius)
- **Background:** Colored backgrounds (e.g., `bg-blue-100`, `bg-green-100`, `bg-purple-100`)
- **Icon Color:** Matching colored text (e.g., `text-blue-500`, `text-green-500`, `text-purple-500`)
- **Hover Effect:** `group-hover:scale-110 transition-transform duration-300`

**Example:**
```tsx
// ✅ DO: Use large feature icon pattern
<div className="flex-shrink-0 w-14 h-14 rounded-xl bg-blue-100 flex items-center justify-center group-hover:scale-110 transition-transform duration-300">
  <svg className="w-10 h-10 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
  </svg>
</div>
```

### **Medium Action Icons (Buttons, Action Items)**
- **Container:** `w-10 h-10` (40px × 40px)
- **Icon:** `w-6 h-6` (24px × 24px) - **CRITICAL: Always use inline SVG, never react-icons**
- **Container Style:** `rounded-lg` (8px border radius)
- **Background:** Colored backgrounds with hover states
- **Icon Color:** Matching colored text (e.g., `text-blue-600` for edit, `text-red-600` for delete)
- **SVG Attributes:** `fill="none" stroke="currentColor" viewBox="0 0 24 24"` with `strokeWidth={2}`
- **Hover Effect:** `hover:scale-110 transition-all duration-300`

**Example:**
```tsx
// ✅ DO: Use medium action icon pattern with inline SVG
<button
  className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-100 hover:bg-blue-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
  title="Edit Item"
  aria-label="Edit Item"
  type="button"
>
  <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
  </svg>
</button>
```

**Media Gallery Grid Buttons (Medium Action Icons):**
```tsx
// ✅ DO: Use inline SVG icons in media gallery grid tiles
<div className="p-4 pt-0 flex justify-end gap-2">
  {/* Edit Button */}
  <button
    onClick={() => handleEditClick(item)}
    className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-100 hover:bg-blue-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
    title="Edit Media"
    aria-label="Edit Media"
    type="button"
  >
    <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
    </svg>
  </button>
  {/* Delete Button */}
  <button
    onClick={() => handleDelete(item)}
    className="flex-shrink-0 w-10 h-10 rounded-lg bg-red-100 hover:bg-red-200 flex items-center justify-center transition-all duration-300 hover:scale-110"
    title="Delete Media"
    aria-label="Delete Media"
    type="button"
  >
    <svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
    </svg>
  </button>
</div>
```

### **Small Inline Icons (Text, Lists)**
- **Icon:** `w-4 h-4` (16px × 16px) or `w-5 h-5` (20px × 20px)
- **No container needed** - inline with text
- **Color:** Inherit text color or use semantic colors

**Example:**
```tsx
// ✅ DO: Use small inline icon pattern
<button className="flex items-center gap-2">
  <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 9l-7 7-7-7" />
  </svg>
  <span>Read More</span>
</button>
```

---

## **Color Palette for Icons**

### **Semantic Colors**
- **Date/Calendar:** `bg-blue-100` / `text-blue-500`
- **Time/Clock:** `bg-green-100` / `text-green-500`
- **Location/Map:** `bg-purple-100` / `text-purple-500`
- **Calendar/Add to Calendar:** `bg-orange-100` / `text-orange-500`
- **View/Details:** `bg-green-100` / `text-green-700` (table buttons) or `bg-white/20` / `text-white` (on colored backgrounds)
- **Copy:** `bg-blue-100` / `text-blue-700`
- **Navigate/External Link:** `bg-green-100` / `text-green-700`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
