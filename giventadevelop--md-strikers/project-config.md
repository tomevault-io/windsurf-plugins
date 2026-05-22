---
trigger: always_on
description: Standard pagination footer styling pattern with Previous/Next buttons and page information display
---


# Pagination Footer Styling Pattern

## **Overview**
This rule defines the standard pattern for pagination footer controls used across admin pages and list components. The pattern includes Previous/Next navigation buttons, page information display, and item count text, all styled consistently with the admin interface design system.

## **Problem Solved**
- **Consistent Pagination UI**: Ensures all pagination footers use the same visual pattern
- **Button Styling**: Standardized Previous/Next button appearance with hover effects
- **Page Information Display**: Consistent page number and item count presentation
- **Accessibility**: Proper ARIA labels and disabled states
- **Responsive Design**: Works across different screen sizes

## **Core Pattern**

### **Pagination Container**
```tsx
// ✅ DO: Use the standard pagination footer container
<div className="mt-8">
  <div className="flex justify-between items-center">
    {/* Previous Button */}
    {/* Page Info */}
    {/* Next Button */}
  </div>
  <div className="text-center mt-3">
    {/* Item Count Text */}
  </div>
</div>
```

### **Previous/Next Button Structure**
```tsx
// ✅ DO: Use the standard pagination button pattern
<button
  onClick={onPrevPage}
  disabled={isPrevDisabled}
  className="px-5 py-2.5 bg-blue-100 hover:bg-blue-200 text-blue-700 font-semibold rounded-lg shadow-sm border-2 border-blue-400 hover:border-blue-500 disabled:bg-blue-100 disabled:border-blue-300 disabled:text-blue-500 disabled:cursor-not-allowed flex items-center gap-2 transition-all duration-300 hover:scale-105 hover:shadow-md"
  title="Previous Page"
  aria-label="Previous Page"
  type="button"
>
  <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M15 19l-7-7 7-7" />
  </svg>
  <span>Previous</span>
</button>
```

## **Key CSS Properties**

### **Pagination Container Requirements**
- **`mt-8`**: Top margin (32px) for spacing above pagination
- **`flex justify-between items-center`**: Horizontal layout with buttons on sides, page info in center
- **`text-center mt-3`**: Centered item count text with spacing (12px)

### **Button Requirements**
- **`px-5 py-2.5`**: Padding (20px horizontal, 10px vertical)
- **`bg-blue-100`**: Light blue background
- **`hover:bg-blue-200`**: Darker blue on hover
- **`text-blue-700`**: Dark blue text color
- **`font-semibold`**: Bold text weight
- **`rounded-lg`**: Large border radius (8px)
- **`shadow-sm`**: Small shadow for depth
- **`border-2 border-blue-400`**: 2px border with medium blue color
- **`hover:border-blue-500`**: Darker border on hover
- **`disabled:bg-blue-100 disabled:border-blue-300 disabled:text-blue-500 disabled:cursor-not-allowed`**: Disabled state styling
- **`flex items-center gap-2`**: Flex layout with icon and text, 8px gap
- **`transition-all duration-300`**: Smooth transitions
- **`hover:scale-105`**: 5% scale increase on hover
- **`hover:shadow-md`**: Medium shadow on hover

### **Page Info Display Requirements**
- **`px-4 py-2`**: Padding (16px horizontal, 8px vertical)
- **`bg-blue-50`**: Very light blue background
- **`border-2 border-blue-300`**: 2px border with light blue color
- **`rounded-lg`**: Large border radius (8px)
- **`shadow-sm`**: Small shadow
- **`text-sm font-bold text-blue-700`**: Small, bold, dark blue text
- **`text-blue-600`**: Medium blue for page numbers

### **Item Count Text Requirements**
- **`inline-flex items-center`**: Inline flex layout
- **`px-4 py-2`**: Padding (16px horizontal, 8px vertical)
- **`bg-blue-50`**: Very light blue background
- **`border-2 border-blue-300`**: 2px border with light blue color
- **`rounded-lg`**: Large border radius (8px)
- **`shadow-sm`**: Small shadow
- **`text-sm text-gray-700`**: Small, gray text
- **`font-bold text-blue-600`**: Bold, medium blue for numbers

## **Icon Specifications**

### **Previous Button Icon (Left Arrow)**
```tsx
<svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M15 19l-7-7 7-7" />
</svg>
```
- **Size**: `w-5 h-5` (20px × 20px)
- **Stroke Width**: `2.5` (thicker for visibility)
- **Color**: Inherits from button text color (`text-blue-700`)

### **Next Button Icon (Right Arrow)**
```tsx
<svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2.5} d="M9 5l7 7-7 7" />
</svg>
```
- **Size**: `w-5 h-5` (20px × 20px)
- **Stroke Width**: `2.5` (thicker for visibility)
- **Color**: Inherits from button text color (`text-blue-700`)

## **Complete Example**

### **Full Pagination Footer**
```tsx
{/* Pagination Controls - Always visible, matching admin page style */}
<div className="mt-8">
  <div className="flex justify-between items-center">
    {/* Previous Button */}
    <button
      onClick={onPrevPage}
      disabled={isPrevDisabled}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
