---
trigger: always_on
description: Standard pattern for admin home button groups with grid layout for navigation buttons in admin pages and sub-pages
---


# Admin Home Button Groups Pattern

## **Overview**
This rule defines the standard pattern for navigation button groups displayed in a grid layout across admin pages and sub-pages. These button groups provide consistent styling, responsive grid layout, and hover effects matching the admin home page design.

## **Problem Solved**
- **Consistent Grid Layout**: Ensures all admin navigation button groups use the same responsive grid system
- **Card-Style Buttons**: Provides consistent card appearance with shadows and rounded corners
- **Icon Standardization**: Provides consistent icon container and sizing (14x14 container, 10x10 icon)
- **Hover Effects**: Standardized hover states with scale transforms and background color changes
- **Responsive Design**: Ensures buttons adapt properly across mobile, tablet, and desktop breakpoints
- **Accessibility**: Proper ARIA labels and titles for screen readers

## **Core Pattern**

### **Grid Container**
```tsx
// ✅ DO: Use responsive grid layout matching admin home page
<div className="w-full mb-8">
  <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
    {/* Button cards */}
  </div>
</div>
```

### **Button Card Structure**
```tsx
// ✅ DO: Use card-style button pattern
<Link
  href="/admin/path/to/resource"
  className="flex flex-col items-center justify-center bg-{color}-50 hover:bg-{color}-100 text-{color}-800 rounded-lg shadow-md p-4 text-xs transition-all group"
  title="Button Label"
  aria-label="Button Label"
>
  <div className="flex-shrink-0 w-14 h-14 rounded-xl bg-{color}-100 flex items-center justify-center mb-3 group-hover:scale-110 transition-transform duration-300">
    <IconComponent className="w-10 h-10 text-{color}-500" />
  </div>
  <span className="font-semibold text-center leading-tight">Button Label</span>
</Link>
```

## **Key CSS Properties**

### **Grid Container Requirements**
- **`w-full`**: Full width of parent container
- **`mb-8`**: Standard margin bottom (32px) for spacing
- **`grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4`**: Responsive grid
  - Mobile: 1 column
  - Small screens (640px+): 2 columns
  - Medium screens (768px+): 3 columns
  - Large screens (1024px+): 4 columns
- **`gap-4`**: Consistent gap between grid items (16px)

### **Button Card Requirements**
- **`flex flex-col items-center justify-center`**: Centers content vertically and horizontally
- **`bg-{color}-50`**: Light background color matching action type
- **`hover:bg-{color}-100`**: Darker background on hover
- **`text-{color}-800`**: Text color matching action type
- **`rounded-lg`**: Medium border radius (8px) for card appearance
- **`shadow-md`**: Medium shadow for depth
- **`p-3`**: Padding (12px) for card content - reduced by 25% from original p-4
- **`text-xs`**: Small text size for labels
- **`transition-all`**: Smooth transitions for all properties
- **`group`**: Enables group hover effects on child elements

### **Icon Container Requirements**
- **`flex-shrink-0`**: Prevents icon container from shrinking
- **`w-11 h-11`**: Fixed icon container size (44px × 44px) - reduced by 25% from original w-14 h-14
- **`rounded-xl`**: Large border radius (12px) for icon container
- **`bg-{color}-100`**: Background color matching button hover state
- **`flex items-center justify-center`**: Centers icon within container
- **`mb-2`**: Margin bottom (8px) for spacing between icon and text - reduced by 25% from original mb-3
- **`group-hover:scale-110`**: Scales up 10% when parent card is hovered
- **`transition-transform duration-300`**: Smooth scale animation

### **Icon Requirements**
- **`w-8 h-8`**: Icon size (32px × 32px) - reduced by 25% from original w-10 h-10
- **`text-{color}-500`**: Icon color matching action type (medium shade)

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-center`**: Centers text horizontally
- **`leading-tight`**: Tighter line height for compact display

## **Color Coding System**

### **CRITICAL: Unique Color Requirement**
- **NO TWO BUTTONS IN THE SAME BUTTON GROUP CAN HAVE THE SAME BACKGROUND COLOR**
- **ALL GRAY COLORS ARE PROHIBITED** - Never use `gray`, `slate`, `stone`, `zinc`, or `neutral` colors for button backgrounds
- Each button in a button group must have a unique, vibrant color to ensure visual distinction
- This requirement applies to all admin pages and subpages that display button groups
- When adding new buttons to a group, ensure the color is not already used by another button in that group
- Standard color assignments:
  - **Admin Home**: Always use `blue` (not gray)
  - **Manage Usage**: Always use `indigo` (not blue, to differentiate from Admin Home)
  - All other buttons: Use unique colors from available palette (green, teal, purple, violet, orange, pink, rose, lime, yellow, fuchsia, cyan, amber, emerald, sky, red, or custom colors)

### **Semantic Colors for Navigation**
- **Blue** (`blue-50/100/500/800`): Admin Home (standard - replaces gray)
- **Indigo** (`indigo-50/100/500/800`): Manage Usage, User Management (standard - replaces blue for Manage Usage)
- **Green** (`green-50/100/500/800`): Manage Events, Calendar Actions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
